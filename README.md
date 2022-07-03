# Data Science Project: Replication of Impact of Stack Overflow Code Snippets on Software Cohesion


## Introduction

The use of Stack Overflow (SO) in software development is a common occurrence, in particular copied code snippets from SO can be seen throughout many projects on Github. Software developers often intuitively seem to know that copying code directly from stack overflow, especially without modifying it, is not necessarily best practice. However it is difficult to know what effect it really has, more than just anecdotally, without studies examining this on a larger scale. Thus it is important to study the effect such copied code snippets have on projects as a whole in order to understand and minimize any possible negative impact of this practice. As many questions related to this had not been adequately answered, the MSR 2019 mining challenge was dedicated to research relating to the evolution and impact of SO [4].

Code snippets from Stack Overflow can be directly copied and pasted or be modified before being added to a development project, thus it is an interesting question as to what extent developers make an effort to modify copied code snippets, and how much of a benefit that effort is. One 2019 challenge paper by Ahmad & Ó Cinnéide [2], examines this topic and specifically the question of “how does copying code snippets from SO affect a code quality over time?” using the metric of code cohesion. Code cohesion is defined as “how closely all the routines in a class or all the code in a routine support a central purpose—how focused the class is”  and is viewed as one heuristic for assessing code quality [8]. We attempt to replicate Ahmad & Ó Cinnéide’s study and further extend it to investigate whether similar results are obtained when using a different dataset. 


## Related Work

In part due to the focus of MSR 2019, there has been a number of papers relating to research on SO, including how developers use SO and the quality of code snippets on the site. There have been comprehensive studies examining the quality of the snippets themselves. We discuss a few notable papers studying this same topic.

Previous papers have used various methods to assess SO code including through examining the toxicity of online code clones on SO by [10]. This paper from Ragkhitwetsagul et al. looks at how copied code snippets can introduce issues of outdated code and code which violates the original software license. Using a mixture of Stack Overflow user surveys and clone detection software, they found that a significant number, 66%, of users surveyed do not check for license conflicts when copying code snippets. Moreover, their results showed 100 out of the 153 identified copied codes were outdated. Although the surveys were on a relatively small scale, this suggests that perhaps there needs to be a heavier focus on reviewing the quality and licensing of SO code before copying it.

The study by Zhang et al. [11] analyses the misuse of API calls in SO code, investigating both the proportion of posts with such misuse and the most common types of misuse. Through the development of an API usage mining technology and specifically targeting Java and Android APIs, Zhang et al. found that 31% of the posts in their data set contain potential misuse. Interestingly, they also observed no correlation between the number of votes on a post and its correct API usage.  

By mining the source code of 22 android apps and using clone detection techniques, Abdalkareem et al. [1] are able to examine the frequency of code reuse from SO and how it affects the number of bugs in the code. In particular, using timestamps to identify code reuse and commit messages to measure the number of bugs in the code, they found that copying and pasting code increases the amount of bugs in a file. This is another alternative approach to measuring the quality of code, but its accuracy relies solely on human bug identification and documentation in commit messages.

In comparison to previous papers, [2] takes a somewhat novel approach by instead assessing the quality of the project before and after incorporating a specific SO code snippet and thereby assessing the impact of the snippet of the project's overall quality rather than just the quality of the snippet itself. This approach in particular lets us more clearly examine the real impact of copying and pasting code. Recently the results of [2] have been used to push forward further research. Namely, Meldrum et al. [9] have done an extensive study on how these code snippets evaluate against many different metrics and cite [2] as part of their motivating research. 


## Methodology

### Dataset and Preparation 

The dataset we use is the SOTorrent dataset [4] which was used as the basis for the MSR 2019 Mining Challenge and is thus the same as in the original paper [2]. This dataset includes a table called PostReferenceGH which links SO posts to GitHub files in which the posts are used or referenced. We use the tool BigQuery from Google to access the database and select our samples [5]. We select samples in the same way as [2], choosing 489 random samples of files which meet our three sample criteria which are as follows. First the files must be in java, as having samples in different languages makes it difficult to get the parameters we will use for our metrics and it is a very popular language. Second, the first commit cannot contain the copied SO snippet since if there is no commit without the SO snippet, we have nothing to compare with the commit containing the snippet. Lastly, the sample must have at least three commits since otherwise we cannot fully assess project cohesion over time in projects with less than three commits. The full process diagram of our workflow can be seen in figure 1.

![image](https://user-images.githubusercontent.com/47286892/177054062-88bf57c8-8817-4880-93e1-164ebddcab78.png)
Figure 1: Process Diagram
