I think it apropos to begin by mentioning that this document is not meant to be exhaustive. I hope this can act as a handy cheatsheet for when that pesky Zscaler ticket lands in your pile, that's my intention here. This is, essentially, my learnings from troubleshooting Zscaler issues for over two years, documented.

This cheatsheet references the study material for [Zscaler Certified Cloud Administrator - Private Access (ZCCA-PA)](https://www.zscaler.com/resources/training-certification-overview) extensively and therefore follows the same structure the study materials do.

- <b>L1 Suggested Troubleshooting Flow</b>

Well, the first thing you'd want to check is whether the end user is even connected to ZPA. That is, verify that the `Service Status` is `ON` and, additionally, depending on how you have configured Zscaler for your organization, the `Authentication Status` is `Authenticated`. It sounds rather obvious, but trust me, this has been the resolution to quite a few tickets in my time so you would want to cover all your bases before you get into the really technical bits.

< placeholder for ZCC image >
