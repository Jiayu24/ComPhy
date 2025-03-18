# Bayesian inference

Today we are going to estimate Bayes phylogenetic tree using popular suite BEAST2 (https://www.beast2.org/), which represents a collection of programs. Download corresponding version on your local machine as well as on HPC. Additionally, in this lab we are going to verify convergence of our Bayes models using program called Tracer v.1.7.2 (https://github.com/beast-dev/tracer/releases). You will need to download Tracer on your local machine.     

Once you download BEAST2, you will find two versions for most of the BEAST2 programs: Graphical User Interface (GUI) and executable (can be found in the ```bin``` directory). For this lab, we will use BEAUti GUI to build input configuration files, and then we will use BEAST executable with this input file. Typically, BEAST input files contain a set of user-specified instructions such as MSA, substitution model parameters, priors, MCMC parameters, etc. to run inference in BEAST.

We will attempt to estimate phylogenetic trees for two datasets: ```greens.fasta``` and ```turtle.fa```, that you can find in our GitHub repo in the ```Labs``` directory.   

### Greens dataset

1) First, lets generate config file in BEAUti for  ```greens.fasta```. Import alignment into BEAUTi by clicking on the "+" button at the bottom of the BEAUTi's ```Partitions``` tab.  

![Fig1](https://github.com/antonysuv/compphylo/blob/main/Labs/figs/Fig1.png)

**Question** : How many taxa and sites does our dataset have?  

8 and 920.

2) Move to the ```Site model``` tab and set up your substitution model. In this run we are going to use GTR+I model.   

![Fig2](https://github.com/antonysuv/compphylo/blob/main/Labs/figs/Fig2.png)

**Question** : From how many substitution models you choose in BEAST2 to run your analysis?   

4

**Question** :  What does "Frequencies" parameter specify here? What are the two main ways you can specify these parameters in phylogenetic inference?     

The frequencies of ATCG in the alignment. Use specific asumption or use the frequencies derived from the input data.

**Question** : Why are we only specifying 6 rate parameters in our models whereas the size of a nucleotide substitution rate matrix Q is 4x4=16? 

We will not count from A to A, C to C, G to G, T to T, and from A to T and T to A, we only count once.

3) Move to the ```Priors``` tab and choose your priors and their hyperparameters.   
   
![Fig3](https://github.com/antonysuv/compphylo/blob/main/Labs/figs/Fig3.png). 

For the Tree prior we are going to use pure birth model (Yule model), which is a special case of Birth-Death (BD) model where death rate parameter is 0. As of note, BD (and hence Yule) model is a continuous-time Markov models.   

**Question** :  How many parameters does Yule model have? What is the prior distribution we are using? What is the domain of this prior? 
3, birth Rate, frequencyParameter and propotional Invariant.   
For birthRate, the initial = [1.0] and the domain is 0 to infinity.
For freqPatameter, the initial = [0.25] and the domain is 0 to 1.
For propotionInvariant, the initial = [0.0] and the domain is 0 to 1.

**Question** :  Why the domain for +I prior is bounded between 0 and 1?         
Because it represent the proportion of invariable sites, for percentage, the domain is between 0 and 1.

4) Move to the ```MCMC``` tab and set up your MCMC chain parameters. 
   
![Fig4](https://github.com/antonysuv/compphylo/blob/main/Labs/figs/Fig4.png). 

Since we are going to run our MCMC for just 100000 generations it is ok to save every 10th iteration parameter in the ```tracelog```. However, usually, a total of 10000 samples in the trace log is sufficient, and having more samples only wastes disk space and makes processing the trace log more cumbersome. So, taking the total number of MCMC steps and divide this by 10.000 makes a good value for logEvery.

In the ```treelog``` we will be storing a tree every 10th generation. 

5) Once all the model parameters and priors are set we can save our config file in the .xml format that we are going to use for tree inference in BEAST2. Click on ```File```, then ```Save as``` and save your xml as ```greens.xml```.  

6) Transfer your ```greens.xml``` to ARC and use it in your BEAST2 run like so: 
```shell
beast greens.xml
```

**Question** : What is the final likelihood value?

-3263.7929435334563

**Question** : How many MCMC tree samples were saved in ```greens-greens.trees```?   

10000

7) Now we are going to summarize the tree posterior sample and print posterior probabilities (PP) for each branch. In Bayesian phylogenetic we assume that reliable inferences exhibit PP>0.95.   
```shell
treeannotator -file greens-greens.trees greens.tre
```

Transfer ```greens.tre``` file to your local machine and visualize it in FigTree program. Navigate to branch labels and then choose to display "posterior". 

**Question** : What are the lowest and highest PPs on your estimated tree? What conclusion you can make about taxonomic relationships based on this inference? 

The lowest PPs is 0.5465, between Anacystis_nidulans and Chlamydomonas clade. The highest PPs is 1.
Rice and tobacco has the closest evolutionary distance compared with other taxa.

**Question** : What is the ancestral branch length of Rice and Tobacco?  What is the 95% High posterior density interval (HPD) for that branch length?   

0.0101 and [0.0065, 0.0141]

8) Now let's investigate whether or BEAST run has reached convergence in Tracer. On that end, transfer  ```greens.log``` file to your local machine and open it in Tracer. To assess convergence for continuous parameters we use The Effective Sample Size (ESS). Spend some time familiarizing yourself with this [criteria](https://revbayes.github.io/tutorials/convergence/).    

**Question** : Based on ESS, which parameters converged and which didn't?  

Tree.height, Tree.treeLength have the ESS at 112 and 113, Some parameters have ESS values around 120, which indicates that the sampling is not fully sufficient.
YuleModel and birthRate have the ESS at 1029 and 796, and they are converged. For posterior, likelihood, prior, treeLikelihood and all rate, frequency parameter are not converged.

**Question** : Do you think increasing the MCMC length will improve your results? To answer this question re-run analysis using 5000000 MCMC iterations and submit Tracer screenshot.

Yes.
![alt text](/fig/image.png)

**Question** : Why the first run of 100000 MCMC draws produced bimodal distribution of the posterior?  

The chain is sill walk around to find the best results.

### Turtle dataset

Repeat BEAST2 analysis with our turtle dataset under JC substitution and 4 gamma categories, i.e. JC+G4. Set the length of MCMC chain to 1000000.

**Question** : Provide a screenshot of your ESS estimates and the final tree with the plotted PPs. Did this analysis produce converged results?      

![alt text](/fig/image-3.png)
![alt text](/fig/image-4.png)
No, based on the results, posterior, likelihood, treelikelihood and gammaShape are not converged.

**Question** : Does it take longer to run BEAST with turtle dataset vs. green dataset? What are the reasons for the differences in scalability?     

Yes, there are 16 taxas and 20820 sites. 

**Question** : Will Bayesian approaches be tractable for large phylogenomic datasets? What do you think are the obstacles towards scalability? List at least 3 and rank them in respect to their effect on scalability.   

No, there are so much data to store. Taxa, Sites, model.

**Question** : Can you propose some remedies that may help improving speed of the Bayesian analyses? Propose at least one approach and explain why it can be useful.

Split the length of MCMC chain into 10 parts and run them at the same time, then combine the results.