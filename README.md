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

![Untitled drawio (2)](https://user-images.githubusercontent.com/47286892/177054079-44f146e8-1ff3-46a9-8808-7741b1f8b143.png)
_Figure 1: Process Diagram_

After selecting initial samples with the .java extension from the PostReferenceGH table in SOTorrent, we used the Github API to then identify which samples met the remaining criteria.  For the final dataset we needed the file version just before the SO snippet was introduced, called the pre-SO snapshot and the file version just after, called the post-SO snapshot. For a select subset of the projects classified as deteriorating (see Results section for more details) we also require the file history after the post-SO snapshot. To obtain needed commits, we used GitHub api to find and obtain all of the files that correspond to the chosen samples starting from the post-SO snapshot to the most recent commit. 

Due to time constraints and underestimating the difficulty of obtaining our complete dataset we were unable to analyze data related to python projects and only obtained a dataset for Java projects as in [2].

### Metrics

Following the choices in [2], we use the metrics Class Cohesion (CC) [6] and Low-level Similarity-based Class Cohesion (LSCC) [3] to measure the code cohesion of the pre- and post-SO snapshots. Both metrics output values in the range of [0,1] where 1 represents perfect class cohesion. 

<img width="400" alt="image" src="https://user-images.githubusercontent.com/47286892/177054132-e03d1046-8bee-4296-a21c-d9a91de08483.png">

CC uses the idea of measuring the similarity between pairs of methods. In terms of the equation Ii represents the set of attributes referenced by method i and k is the total number of methods in the class. The fraction in absolute values is the similarity between method i and method j, calculating the ratio of attributes they have in common over the number of attributes they reference in total. Then taking the summation of all the similarities between each pair of methods in the class gives our total measure of cohesion.

<img width="480" alt="image" src="https://user-images.githubusercontent.com/47286892/177054155-e8bdf3eb-9f44-4ca2-847d-7bbc32923007.png">

In LSCC, k represents the number of methods in the class while l is the number of attributes. It first categorizes extreme cases where it is clear the class is either very cohesive or not cohesive at all. To evaluate classes which lie somewhere in between, it uses the Method Attribute Reference (MAR) to calculate the ratio of summation of common attributes to the total number of methods and attributes in the class. The MAR of a class is an k by l array where each row corresponds to a method and each column corresponds to an attribute. Then entry (i,j) of the MAR is 1 if method i contains attribute j and 0 otherwise. Each xi in the LSCC formula corresponds to the sum of the ith column of the MAR, which means the number of methods which use attribute i. For ease of computation we store the MAR in our code as a 1 by l array of the already summed column values. An example MAR from [3] where the matrix was first defined is shown in table 1 to further clarify the concept. This is an example corresponding to a hypothetical example where each print_ represents a method of the same name and the class has the attribute x,y, and z. 


<img width="400" alt="image" src="https://user-images.githubusercontent.com/47286892/177054179-d0702b06-cae6-438b-8e81-385b961535d5.png">
Table 1: an example MAR

### Research Questions

Our project seeks to answer two questions about how Stack Overflow code snippets affect the cohesion of the recipient classes. These questions follow directly from [2] in the hopes we can replicate their results or identify why there could be discrepancies.

RQ1: Is there a difference in cohesion between the pre-SO class and post-SO class? 
RQ2: When there is a decrease in cohesion between pre-SO and post-SO snapshots, does the difference stay consistent as the class evolves? 

Knowing the results to these questions would be of particular importance when considering whether or not to reuse code snippets from Stack Overflow. For example if cohesion is known to drop when introducing snippets then a developer may either refrain from using snippets or take extra precautions and adjustments in order to maintain their code quality. RQ2 helps developers gauge whether or not an initial drop in cohesion will self-correct over time or if active action is required.


## Results

After we finished running our analysis software, we needed to manipulate the resulting csv file into a Tidy format in order to properly categorize the metrics. Once this was done we were able to import the data into python to create visualizations using the same criteria as the paper; varying, constant, deteriorating, and improving for the change between pre-SO snapshot and post-SO snapshots, along with fully recovered, partially recovered, and not recovered for the change from post-SO snapshots throughout the rest of the file history [2].

We classify the change values for the cohesion metrics of the pre- and post- SO snapshots as either constant, meaning the value does not change, or varying, meaning the cohesion does change. Of those classified as varying, projects whose values are lower in the post-SO snapshot can be called deteriorating and those whose values are higher in the post-SO snapshot can be called improving as seen in figures 1 and 2. Using these classifications, RQ1 can be assessed directly through the results of the CC and LSCC metrics on the SO snapshots. 

When comparing our graphs with those of the [2], we obtained surprisingly similar results however we found both a higher percentage of constant (45.3% vs 39% for LSCC and 44.4% vs 42% for CC) and improving (21.8% vs 19% for LSCC and 20.9% vs 16% for CC) cases and thus subsequently a decrease in the deteriorating (32.9% vs 42% for LSCC and 34.7% vs 42% for CC) cases.

<img width="380" alt="image" src="https://user-images.githubusercontent.com/47286892/177054229-e36d19ec-7b19-4776-81ea-1cf4800b6bfd.png">
Figure 2: categorical breakdown of overall CC changes from pre-SO snapshots to Post-SO snapshots

<img width="380" alt="image" src="https://user-images.githubusercontent.com/47286892/177054243-a5beef03-97d3-4214-ad8a-529079445f22.png">
Figure 3: categorical breakdown of overall LSCC changes from pre-SO snapshots to Post-SO snapshots

We further analyse those projects classified as deteriorating to understand if they ever improve again. Intuitively, we classify these projects as fully recovered, partially recovered, or never recovered based on if the project’s cohesion value ever returns to the level of its pre-SO snapshot. Specifically, fully recovered projects return to cohesion levels at or above their pre-SO snapshot and partially recovered projects reach at least half of their previous value which can be seen in figure 4. Answering RQ2 requires these further classifications as we need to compare the code cohesion values of the pre-SO snapshots to later versions in the project’s history. 

Looking at the longer term history of those which fell into the decreasing category for RQ1, our results differ quite a bit from [2]. They had found that, overwhelmingly, the cohesion of these decreasing classes never got back to the level it was before the SO code. However we found that almost half (approximately 49% for both metrics) of all deteriorating classes partially recovered, meaning that they are at least half of the original cohesion value, as did never recover. Furthermore, our results show roughly 9% or deteriorating cases fully recover either metric while [2] shows 18% fully recover.

<img width="450" alt="image" src="https://user-images.githubusercontent.com/47286892/177054264-d6b6b427-2759-452e-9c09-d59d47c074c4.png">
Figure 4: Overall LSCC and CC recovery status of the deteriorating cases from pre-SO snapshot to the latest commit

Finally we looked at a few individual files to illustrate how LSCC and CC metrics change over time. We chose 3 random files and graphed the metric changes vs the commit number.  Commit 0 is the pre-SO snapshot and commit 1 is the commit where the SO snippet was added. These visualizations can be seen in figures 5, 6, and 7. One interesting thing to note is we happened to get an example of each breakdown of metric changes; Figure 5 shows the metrics improving, figure 6 shows a sharp deterioration and figure 7 shows a constant value from pre-SO to post-SO snapshots.

## Discussion and Limitations

From our results and those of [2], we can see a large number of cases in which LSCC and CC decline immediately following the introduction of a SO snippet. Although some threats to validity exist as discussed later, it is still reasonable to assume that caution should be used when utilizing SO snippets in projects if maintaining LSCC and CC is desired. This may be especially important in more critical software projects where decreases in code quality can result in losses. Something that remains less clear is the recovery of each metric in cases where they deteriorated. While both studies show a low percentage of fully recovered projects, we see a much larger percentage of partially recovered projects vs [2].

In terms of ethics, there are always information privacy concerns that need to be taken into account when doing data mining. The paper “Ethical Mining: A Case Study on MSR Mining Challenges” specifically about this sort of mining makes the potential ethical issues of this paper (and the one we are replicating) particularly clear [7]. Although we used the SOTorrent Database to get the initial matches, what we really ended up handling the most was data from Github. This included the different file versions and commit information. In particular with the sort of github data we ended up analysing, it’s very easy to trace back data to specific profiles and from there identify the actual author. To this end we take care to not use any specific usernames or identifiers in our paper. Even in the individual timeline graphs, we name them based on the larger project the files were committed to (which each have dozens or hundreds of files and commits) rather than the file names themselves which are much more revealing in terms of authorship.

<img width="500" alt="image" src="https://user-images.githubusercontent.com/47286892/177054306-1a6ad55b-5ca8-401b-9ce8-6b6d8a3478bf.png">
Figure 5: appinventor-sources project metric change vs commits

<img width="500" alt="image" src="https://user-images.githubusercontent.com/47286892/177054311-a6287380-1444-4b35-a336-b9ba0a81c609.png">
Figure 6: netty project metric change vs commits

<img width="500" alt="image" src="https://user-images.githubusercontent.com/47286892/177054313-0f2c11b5-40a0-4315-9d59-b925e46a6861.png">
Figure 7: intellij-community project metric change vs commits

Next we discuss a few potential limitations and threats to validity of our study which mainly are perpetuated from [2].  Firstly, we cannot be certain that when assessing the cohesion of the pre-SO and post-SO snapshots the SO code snippet is the only thing affecting code cohesion. There could be other confounding factors/updates of the code although with a large enough sample size we can expect to mitigate this issue somewhat.  This remains a large threat to validity though as the timeline graphs in figures 5-7 do show that cohesion changes happen throughout a project’s lifetime regardless of adding SO snippets.

Additionally, while cohesion is a useful and easily comparable measure of a project, it is not clear that cohesion itself is enough to make a definitive assessment of a code’s level of quality. Although these are popular metrics to assess code quality, they are definitely not always reliable indicators of quality as they only take into account class and instance variable to method ratios. For example the LSCC value of a class with only one method is always 1, which represents perfect cohesion. The hope with these metrics is that when they are applied to enough samples, that they aren’t perfect indicators of quality, they can show the overall trend.

Limitations also extend from the database as SOTorrent only links to projects hosted on Github. While we replicated the approach used to sample the database, the samples chosen were a random distribution and thus not the exact same as the ones used in [2]. These factors and the difference in results between our study and the original suggest the samples used may not be generalizable in or outside of this domain. That the results may not generalize outside of the domain is a common threat to validity in many studies based on studying data from Stack Overflow or Github.

## Conclusion
In this study, we replicated the approach taken in [2] to see the effects of introducing SO snippets to existing Java projects. The main goal was to answer RQ1 and RQ2 while also seeing if we would get the same results as in [2]. Through figures 1 and 2 we can clearly see a difference in LSCC and CC which answers RQ1 roughly in line with the results obtained in [2]. Then in figure 3 we see a difference from [2] where we found more than half the deteriorating cases either recover fully or partially after the initial decrease, while the findings in [2] show roughly 75% of cases never reached either threshold. Deriving the answer to RQ2 from these results shows that in our study the decrease does not remain consistant while in [2] the decrease generally stays similar.

Future studies that could expand upon these findings could include expanding the types of projects to see if specific languages behave differently, analyzing more samples to better approximate the overall population, or using projects that are not derived from Github to better generalize the results. Based on the findings of future studies developers can be more informed about the potential risks (if any) of introducing SO snippets into their projects and what level of modification of the snippets would be necessary to mitigate any risks. Lastly, expanding the study scope to analyze other qualities that may be impacted from introducing snippets may be of concern, such as security. These can help keep projects both healthy and improving by appropriately utilizing SO snippets.

## References

[1] R. Abdalkareem, E. Shihab, and J. Rilling, “On code reuse from stackoverflow: An exploratory study on android apps,” Information and Software Technology, vol. 88, pp. 148–158, 2017. 

[2] Ahmad, Mashal & Ó Cinnéide, Mel. (2019). “Impact of stack overflow code snippets on software cohesion: a preliminary study”. 10.13140/RG.2.2.14791.75688. 

[3] J. Al Dallal and L. C. Briand, “A precise method-method interaction based cohesion metric for object-oriented classes,” ACM Transactions on Software Engineering and Methodology (TOSEM), vol. 21, no. 2, p. 8, 2012.

[4] S. Baltes, C. Treude, and S. Diehl, “SOTorrent: Studying the origin, evolution, and usage of stack overflow code snippets,” in Proceedings of the 16th International Conference on Mining Software Repositories (MSR 2019), 2019.

[5] “BigQuery,” https://cloud.google.com/bigquery, 2010.

[6] C. Bonja and E. Kidanmariam, “Metrics for class cohesion and similarity between methods,” in Proceedings of the 44th annual Southeast regional conference. ACM, 2006, pp. 91–95.

[7] Nicolas E. Gold and Jens Krinke. 2020. “Ethical Mining: A Case Study on MSR Mining Challenges,” In Proceedings of the 17th International Conference on Mining Software Repositories (MSR '20). Association for Computing Machinery, New York, NY, USA, 265–276. DOI:https://doi.org/10.1145/3379597.3387462

[8] S. McConnell and S.M. McConnell. 1993. “Code Complete: A Practical Handbook of Software Construction”. Microsoft Press

[9] S. Meldrum, S.A. Licorish, C.A. Owen, B.T.R. Savarimuthu. “Understanding stack overflow code quality: A recommendation of caution,” Sci. Comput. Programm., 199 (2020), 10.1016/j.scico.2020.102516

[10] C. Ragkhitwetsagul, J. Krinke, M. Paixao, G. Bianco, and R. Oliveto, “Toxic code snippets on stack overflow,” arXiv preprint arXiv:1806.07659, 2018.

[11] T. Zhang, G. Upadhyaya, A. Reinhardt, H. Rajan, and M. Kim, “Are code examples on an online Q&A forum reliable?: A study of API misuse on stack overflow,” in Proceedings of the 40th International Conference on Software Engineering, ser. ICSE ’18. New York, NY, USA: ACM, 2018, pp. 886–896.










