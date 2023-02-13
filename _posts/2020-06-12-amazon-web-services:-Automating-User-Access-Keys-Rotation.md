Rotating user access keys on a regular basis is an important component of the [security best practices for IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#rotate-credentials). And while something similar to the auto-rotation feature for KMS would be a welcome addition to IAM, it is easy to see why that isn't supported yet.

Say you set up an automation in place that can rotate IAM user access keys. Or, better still, AWS announces support for auto key rotation on IAM. How do you update the credentials file (`~/.aws/credentials`) on a user's machine to reflect the new keys? And, if your company has terrible security policies, how do you update it everywhere you have called it in code?

So, achieving fully automated end-to-end key rotation for IAM might not be feasible. How far can we go using code, though? Let's find out.

EDIT: You might also want to refer to [this AWS documentation](https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/automatically-rotate-iam-user-access-keys-at-scale-with-aws-organizations-and-aws-secrets-manager.html) and [the corresponding scripts](https://github.com/aws-samples/aws-iam-access-key-auto-rotation) for additional prescriptive.

We begin by iterating through all IAM users we've got in our account. Something like the following should be good starting point:

```
for user in $(aws iam list-users --output text | awk '{print $NF}'); do
```

Now, we need to define a threshold for keys in our account. How old is too old? Let's stick with 90 days as it is the standard for most ISO certifications. It must be noted though that there is a specific pattern in which AWS works with dates and we would need to adhere to that, if want to make the most of our script. Consider the following output of a test `aws iam list-access-keys --user-name Alice` run:

```
{
    "AccessKeyMetadata": [
        {
            "UserName": "Alice",
            "Status": "Active",
            "CreateDate": "2013-04-03T18:49:57Z",
            "AccessKeyId": "AKIAI44QH8DHBEXAMPLE"
        }
    ]
}
```

See the format in which the value for `CreateDate` is returned? That is what we need achieve. Let's experiment with `date` until we get what we would need.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ date
Mon Jun 12 12:32:56 IST 2020
┌──(thatvirdiguy㉿kali)-[~]
└─$ date -d "90 days ago"
Tue Mar 14 12:33:06 IST 2020
┌──(thatvirdiguy㉿kali)-[~]
└─$ date -d "90 days ago" +%Y-%m-%dT%H:%M:%SZ
2020-03-14T12:33:18Z
```

We can use this command to define a "`dateThreshold`" variable in script. Something like the following should work:

```
dateThreshold=$(date -d "90 days ago" +%Y-%m-%dT%H:%M:%SZ)
```

Now, let's get the "`CreateDate`" for all active keys corresponding to every IAM user in our account. Pipe the standard [AWS CLI](https://docs.aws.amazon.com/cli/latest/reference/iam/index.html) output to some [jq](https://stedolan.github.io/jq/manual/) magic and we get:

```
dateKeys=$(aws --profile iam list-access-keys --user $user | jq -r '.AccessKeyMetadata[]? | if .Status == "Active" then .CreateDate else empty end' 2>/dev/null)
```

Now we get to the tricky bit. How do we compare, and do arthimatic operations, on two dates in computer-speak? The easiest way to do this would be to convert these dates into numbers, of course, and this is precisely what we would be attempting.

```
┌──(thatvirdiguy㉿kali)-[~]
└─$ dateToday="2020-03-14T12:33:18Z"
┌──(thatvirdiguy㉿kali)-[~]
└─$ date -d `echo $dateToday | awk -F 'T' '{print $1}'` +%s
1584124200
```

We have essentially converted the date into seconds since [epoch](https://en.wikipedia.org/wiki/Epoch_(computing)) here. Doing this for the `CreateDate` for every active IAM user key in our account and the `dateThreshold` we have defined would look something like the following:

```
for dateK in ${dateKeys}; do
  if [[ $dateK != "" ]]; then
    start=$(date -d `echo $dateK | awk -F 'T' '{print $1}'` +%s)
    end=$(date -d `echo $dateThreshold | awk -F 'T' '{print $1}'` +%s)
```

Then, it is only the matter of doing some basic maths to check if a key is over the defined thershold:

```
diff=$((($end-$start)/(60*60*24)))
  if [[ $diff -gt "90" ]] ; then

```

And if it is, get the `AccessKeyId` (refer to the above output of that `iam list-access-keys` command) this time around:

```
key=$(aws iam list-access-keys --user $user | jq -r --arg dateK "$dateK" '.AccessKeyMetadata[] | if .Status == "Active" and .CreateDate == $dateK then .AccessKeyId else empty end')
```

Here's where it gets interesting. AWS only allows a maximum of two access keys per user (refer to [this documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_iam-quotas.html)) so the next commands in our scripts will need to work around that. If the user already has two access keys, we will delete before we create—

```
if [[ $keyCount == "2" ]]; then
  # if user has two keys already, delete before create--
  echo "$user has hit the AWS limit of two access key per IAM user. Deleting (Access Key ID: $key) before creating..."
  aws iam delete-access-key --access-key-id $key --user-name $user
  echo "Creating a new access key..."
  aws iam create-access-key --user-name $user
```

—and if they don't, we will first deactivate their current key before creating a new one for them.

```
  # --else follow the recommended procedure of making the current key inactive
  echo "Setting current access key ($key) as Inactive..."
  aws iam update-access-key --access-key-id $key --status Inactive --user-name $user
  echo "Creating a new access key..."
  aws iam create-access-key --user-name $user#
```

But, how do we check for the number of keys assigned to a user? Some good ol' fashioned AWS CLI + jq program magic.

```
keyCount=$(aws --profile pl-internal iam list-access-keys --user $user | jq '.[]? | length')
```

And that's it! You can, of course, setup SES and define a custom template to send a mail to the user once their acess key is rotated, but I'll leave that you. Refer to [the documentation](https://docs.aws.amazon.com/cli/latest/reference/ses/index.html) as always.
