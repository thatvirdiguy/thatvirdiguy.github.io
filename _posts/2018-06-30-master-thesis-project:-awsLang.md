---
title: AWSLang
---

AWSLang is probablistic threat modeling and attack simulation language for the AWS environment. Based on [coreLang](https://github.com/pontusj101/coreLang), it is a domain specific language (DSL) created with [MAL (the Meta Attack Language)](https://github.com/pontusj101/MAL).

This was my master thesis project and is presented here as is. You can read the full report [here](https://kth.diva-portal.org/smash/record.jsf?dswid=-6474&pid=diva2%3A1330746&c=3&searchType=SIMPLE&language=en&query=AWSLang&af=%5B%5D&aq=%5B%5B%5D%5D&aq2=%5B%5B%5D%5D&aqe=%5B%5D&noOfRows=50&sortOrder=author_sort_asc&sortOrder2=title_sort_asc&onlyFullText=false&sf=all) and the corresponding technical bits [here](https://github.com/thatvirdiguy/awsLang). 

Abstract from the aforementioned report:

> Attack simulations provide a viable means to test the cyber security of a system. The simulations trace the steps taken by the attacker to compromise sensitive assets within the system. In addition to this, they can also estimate the time taken by the attacker for the same, measuring from the initial step up to the final. One common approach to implement such simulations is the use of attack graph, which trace the various dependencies of every step and their connection to one another in a formal way.
>
> To further facilitate attack simulations and to reduce the effort of creating new attack graphs for each system of a given type, domain-specific languages are employed. Another advantage of utilizing such a language is that they organize the common attack logics of the domain in a systematic way, allowing for both ease of use and reuse of models. MAL (the Meta Attack Language) has been proposed by Johnson et al. to serve as a framework to develop domain-specific languages. My work is based upon the same.
> 
> This thesis report presents AWSLang, which can be used to design IT system models in context to the AWS (Amazon Web Services) environment and analyse their weaknesses. The domain specifics of the language are inspired from and based on existing literature. A Systematic Literature Review (SLR) is performed to identify possible attacks against the elements in an AWS environment. These attacks are then used as groundwork to write test cases and validate the specification.

If you want to take a look at the slides I used to present my thesis to the examiner, please <a href="https://thatvirdiguy.github.io/files/AmandeepSinghVirdi_AWSLang_ThesisPresentationSlides.pdf" rel="noopener" target="_blank" >click here</a>.

**Note**: "AWSLang" was later rechristened as "awsLang", a new and improved version, that later became the foundation for [securiCAD Vanguard](https://foreseeti.com/securicad-vanguard-for-aws/). That new version should be preferred for basing off any further research, since we ended up not only covering a lot more AWS services, but also polishing things over for the new iteration.
