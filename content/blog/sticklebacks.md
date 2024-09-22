---
title: 'DNA fragility in stickleback fish -- a summary of one of my favourite papers of all time'
author: 'by James Lingford'
date: 2024-09-22T14:56:23+10:00
draft: true
toc: false
math: true
tags: []
categories: []
---

The following is a twitter thread I wrote back in January 2019 summarising an amazing paper I had recently read in Science.
The twitter thread ended up becoming quite popular, and it was (if I do say so myself) an example of good science communication.
I deleted my twitter some time ago, but I think the thread I wrote deserves to live on.
I was able to dig it up from the Wayback Machine, so what follows is how it appeared on twitter but with some minor edits and improvements to clarity.
The paper is a major inspiration for me as to what good science looks like.

## DNA fragility in the parallel evolution of pelvic reduction in stickleback fish

There is an amazing evolutionary biology paper out this week in Science: [DNA fragility in the parallel evolution of pelvic reduction in stickleback fish](https://www.science.org/doi/10.1126/science.aan1425), by Kathleen Xie and coauthors.

![pic1](/images/fish/p1.jpg)

Before we dive into the paper, let's first look at what the big picture is. 
We begin with the observation that numerous species exhibit rapid and repeated bouts of adaptive evolution to new environments [CITE].
Furthermore, geographically isolated populations of the same species have been observed with the same phenotypic adaptations (i.e. parallel evolution) [CITE].
This is the case with stickleback fish, which has lost pelvic fins multiple times as marine populations have colonized and adapted to freshwater environments (see [Jones et al. (2012) Nature](https://www.nature.com/articles/nature10944)).
Pelvic fins in marine sticklebacks are believed to be adaptations for detering predators.
But freshwater sticklebacks lack pelvic fins. Why?
The dominant hypothesis is that freshwater environments are more nutrient scare compared to marine environments, thus investing in costly pelvic fin development would be detrimental to sticklebacks survival in this context.

![pic2](/images/fish/p2.jpg)
*Figure 1 from [Jones et al. (2012) Nature](https://www.nature.com/articles/nature10944)*

How is this evolutionary adaptation so rapid and repeatable?
What we learned in school is that mutations are rare, random, and usually deleterious.
So how can these stickleback fin mutations be common, repeatable, and (given it’s the right environment) beneficial?
Previous work from David M Kingsley’s lab has shown that a gene, “Pitx1”, controls pelvic fin size.
Freshwater sticklebacks have low or no Pitx1 expression (hence a lack of pelvic fins).
Interestingly, they couldn’t find any mutations in the Pitx1 protein sequence, so the mutations that are ablating Pitx1 expression must be located somewhere else in the genome.
A likely suspect is mutation in regulatory DNA sequences (like promotors or enhancers) that are regulating Pitx1 expression.

Indeed, [Shapiro et al. (2004)](https://www.nature.com/articles/nature02415) hypothesised that mutations in regulatory DNA sequences controlling Pitx1 expression (ie. promotors & enhancers) were responsible for this rapid & repeatable evolution.

A big implication here is that mutations at regulatory DNA sequences could lead to dramatic phenotypic changes in some tissues but not others. 
This could enhance the “evolvability” of some traits, since XXX
The same lab previously identified the enhancer responsible for controlling Pitx1 expression in pelvic fin formation: “Pel”. Deletion of the Pel enhancer results in loss of Pitx1 expression and loss of pelvic fins ([Chan et al. (2010) Science](https://www.science.org/doi/full/10.1126/science.1182213)).

![pic3]
*Figures 2 and 3 from [Chan et al. (2010) Science](https://www.science.org/doi/full/10.1126/science.1182213). Deletion of the Pel enhancer ablates growth of the pelvic spines, and injection of a transgene with the Pel enhancer and Pitx1 gene rescues pelvic spine growth*

Curiously, this region of the genome has an unusually high tendency to acquire deletion mutations.
![pic4]

But questions remained. 
Why are frequent deletion mutations localised at the Pel enhancer?
Aren't mutations--especially large deletion mutations like the one here--rare and random? 
How is this happening so rapidly in the wild? 
Enter this new paper by Xie et al. (2019).
The authors had observed in prior work that “Pel enhancer sequences show high predicted helical twist flexibility" which they note is "a DNA feature associated with delayed replication and fragile site instability.”
Thus, they hypothesised that these “special DNA features may shape adaptive variation at the Pitx1 locus”.

If Pel sequences do have unusual DNA features, they should migrate differently on 2D electrophoresis gels. 
So they made plasmids containing Pel sequences from both freshwater & marine sticklebacks. 
And indeed, marine vs freshwater DNA migrates differently on 2D gels.

![2dgels](/images/fish/p3.jpg)

Pel DNA from marine sticklebacks produce a “kink” on 2D agarose gels which is absent in the freshwater variants (which have deletions).
This “kink” is reminiscent of 2D agarose gels run with a form of DNA known as Z-DNA.
The common structural form of DNA is known as B-DNA.
Meanwhile, Z-DNA is simply DNA but with an different structure than usual.
The differences in the atomic structure between B-DNA and Z-DNA causes them to travel at different rates through a 2D agarose gel.

![pic]

Looking at the sequence, Pel contains a lot of TG-repeats (see attached figure)

![pic]

In previous work, long stretches of alternating pyrimidine/purine repeats have been shown to form Z-DNA: pnas.org/content/80/7/1…

![pic]

Z-DNA has been previously shown to be more unstable in vivo (possibly a result of backbone strain from its unfavourable structure).
This leads to chromosomal fragile sites, DNA breaks, and erroneous DNA repair ncbi.nlm.nih.gov/pmc/articles/P…
They tested this idea for themselves by inserting TG-repeats (but no other Pel sequences) into a plasmid.
The TG-repeats alone were enough to recapitulate the Z-DNA “kinks” on 2D gels.

![pic]

Could explain why Pel is so prone to deletion?
To answer that question, they tested the effect of Pel sequences on chromosome stability.
They did this by measuring the rate of DNA double-strand breaks in yeast artificial chromosomes with selectable markers.

It works like so: every time the yeast replicates its genome to divide, the DNA of Pel could break if it is unstable. If it breaks, the gene URA3 is lost. URA3 converts the compound 5-FOA into a toxic compound that kills yeast.
Therefore, by counting the number of yeast that survive on media with 5-FOA, you get a readout of how many DNA breaks occurred.
And whaddya know? Marine Pel was highly prone to DNA breaks every time the cell undergoes division. Freshwater Pel was similar to the control in numer of DNA breaks per cell division.

Interestingly, inverting the Marine Pel sequence resulted in a dramatic reduction in the number of DNA breaks per cell division

![pic]

Why does inverting the Pel sequence alter DNA breakage? Changing TG-repeats to CA-repeats should still form Z-DNA?
Well, apparently, it’s not just Z-DNA, it’s TG-repeats when they are on the lagging-strand during eukaryotic DNA replication! As seen when they reversed the orientation for the origin of replication.

![pic]

And the longer the TG-repeat, the more DNA breaks that occur.

![pic]

Why are TG-repeats so much more unstable when they are on the on the lagging strand and not the leading strand of a DNA replication bubble? That is beyond the scope of this paper, but the answer might have to do with secondary structure formation from TG-repeats
What kind of secondary structure could TG-repeats make?

The authors don't say, but I speculate that they could form G-quadruplexes (of the unimolecular bulged variety). G-quadruplexes are implicated in genomic instability. nature.com/articles/nrm.2…

Secondary structure formation on lagging strand have also been implicated in DNA breaks nature.com/articles/natur… (this is an older review, so if anyone has anything more recent, please send it my way :)

But maybe this DNA break phenomenon they are seeing is just a weird artefact of yeast? Well, they also tested the effect of TG-repeats (and CA-repeats as a control) in mammalian cells.
pnas.org/content/103/8/…
sciencedirect.com/science/articl…


The experiment works by using supF, which is a tRNA in *E. coli* that incorporates a Tyrosine at a UAG amber stop codon. supF allows read through of a lazZ gene containing a nonsense amber codon. Blue/white colony selection can be performed. Mutations to supF form white colonies.

Combining these results, a molecular mechanism becomes clear: TG-repeats in the Pel sequence result in deletion mutations during DNA replication due to the fragility. But it only works if the direction of DNA replication is correct (with TG-repeats on the lagging strand).
But the question remains: what is going on in the stickleback genome? is DNA replication really going in the right direction? If it isn’t, then this mechanism can’t be used to explain stickleback adaptive evolution.
Answering this question isn’t as easy as it first sounds.
Unlike yeast & E.coli, there are no strong consensus sequences for replication origins in metazoans.
Thankfully, nascent strand sequencing (NS-seq) can pinpoint replication origins
nature.com/articles/nrg.2… pnas.org/content/107/1/…

To do NS-seq, they divided stickleback embryos into single cells and did FACS sorting to separate cells based on what stage of the cell cycle they were in.
They captured S-phase cells “in the act” of replicating their DNA. en.wikipedia.org/wiki/Cell_cycl…

They could then do massively parallel sequencing on S-phase cells. Nascent strands formed at replication origins generate more sequencing reads and greater read depth. Regions of high depth therefore correspond to replication origins

And whaddya know? The replication origin nearest to Pel on the chromosome will extend replication in the direction that promotes Pel fragility (i.e. TG-repeats on the lagging strand).

Taking it one step further, the authors used CRISPR/Cas9 to make cuts flanking Pel in stickleback embryos.
DNA repair machinery must have a hard time handing the TG-repeats too, because deletions occurred and the Pel(CRISPR)/Pel(KO) progeny exhibited smaller/no pelvic fins

With all this experimental data in hand, the authors then ask the big question: “Could elevated mutation rates contribute to reuse of Pel deletions in parallel evolution?” 
To answer that, they used population genetics modelling.
Point mutations occur at low rates (~10^−9 per site per generation) and are unlikely to fix in populations, even when conferring a selective advantage.
But fragile sites have higher mutation rates (~10^−5 per site per generation).

“The combined effects on both the “arrival of the fittest” and the “survival of the fittest” may explain why recurrent Pel deletions are the predominant mechanism for evolving stickleback pelvic reduction”
The authors then speculate on how general this fragile-site-driven-evolution could be for other traits in sticklebacks and humans.
They note that sequences associated with fragility are present at 1000's of other positions in the SB genome. Furthermore, they observe that “TG-repeats are enriched in other loci that have undergone recurrent ecotypic deletions during marine-to-freshwater stickleback evolution”

The human genome also has numerous fragile sites, and they note that “nearly half of currently known mutations underlying adaptive traits in modern humans also appear to be produced by mechanisms with elevated mutation rates”

The authors speculate that TG-repeat-associated-fragility – other similar but unknown mechanisms – could explain how “migration of modern humans out of Africa occurred with relatively small populations adapting to new environments in 3000 generations or fewer”
This molecular mechanism that Xie et al. report is truly remarkable. The implication is that it’s not just TG-repeat-associated-fragility at work, but several other mechanisms that can “tune” the evolvability of phenotypic traits, such as...
- TG-repeat expansion/contraction (tunes the fragility & rate of mutation)
- sequence rearrangements (turns mutation “on/off” at a loci)
- change in DNA replication direction (turns mutation “on/off” at a loci)
The further implication is that vertebrate genomes could have several more unknown “tricks” to modulate evolvability of specific loci. Protein coding sequences are “protected”, but loci that regulate the temporal + special expression of proteins are open for evo “experimentation”
For more on this topic, read @SeanBiolCarroll's seminal review: cell.com/cell/fulltext/… #openaccess
SUMMARY: Xie et al. uncover a unique mechanism for “evolvability”. It depends on...
1) TG-repeats flanking an enhancer regulating a gene for a phenotype
2) Exploiting the inherent fragility of TG-repeats during DNA replication (+direction)
3) Fragility leading to DNA breaks
4) DNA repair processes repairing the breaks but deleting the enhancer, dramatically altering a phenotype
5) This is supported by population genetic modelling: showing that it's possible for vertebrates to rapidly adapt to new environments despite small population sizes.
OUTSTANDING QUESTIONS:
- what is the biochemical/biophysical basis for direction dependent TG-repeat fragility?
- when is the Pel loci open to mutation? When is it not?
- how much of the phenotypic variation we see in natural pops is due these type of de novo mutations?
*promoter
Edit: unnecessary use of “de novo” (been reading too many David Baker papers :P )
*lacZ
*Forgot to mention: mammalian cells are transfected and the plasmid is allowed to replicate with cell div. Plasmids are then miniprepped, transformed into E.coli with lacZ, and plated on X-gal plates. Deletion rate to supF can be read out as a function of no. of white colonies
*handling
Important detail I forgot to include in the main thread: previous work has shown that transcription can promote DNA fragility through the formation of R-loops: cell.com/molecular-cell…
∴ the authors reverse URA3 transcription direction to determine this impacts fragility.
*if this impacts fragility... they don’t find it does.
POSTSCRIPT
Thank you everyone! The response to this has been overwhelming! I hypothesised that there was a latent hunger in the twitter science community for paper breakdowns. That hypothesis still stands. This thread even reached non-scientists – something I never expected.
One limitation of this paper is that DNA breaks in TG-repeats are not *directly* observed in stickleback embryos. It is however, inferred from multiple lines of evidence (prevalence of deletions in wild SB populations, yeast & mammalian cell work, correct ori direction in vivo...
...CRISPR) & is consistent with previous work on DNA fragility & evolutionary developmental biology.
Gathering this direct evidence of DNA breaks during cell division is a big technical challenge for the future. Something like FACS on lots of S-phase stickleback embryos -> immunoprecipitation for gH2AX (binds dsDNA breaks) or other proteins in DNA damage repair (credit: @pjie2)
A benefit of making this thread was all the discussion it sparked. For example, as an alternative to my hypothesis of G-quadruplex driven fragility, @wtpowell kindly brought my attention to the phenomenon of R-loop driven fragility: ncbi.nlm.nih.gov/pubmed/25435140 #openaccess
“The region [could be] transcribed [forming] R-loops (DNA:RNA hybrid with single DNA strand), which would explain the TG vs CA difference… They depend on the formation of G-rich RNA to form; transcribe the reverse (C-rich RNA) & they do not form…" - @wtpowell
Furthermore: "R-loops have been shown at replication origins and at class-switch and hypermutation sites for immunoglobulin….” - @wtpowell
FINAL THOUGHTS
It was a joy reading this beautiful paper. Additionally, it was a rewarding experience trying to disassemble it, examine each part (a lot of which I knew little about prior to reading), & reassembling it in my own words for a general audience.
Take it all with a pinch of salt though. I'm no stickleback expert. Even now, I know I’ve barely scratched the surface of this field.

So, next time you come across a paper that you find interesting, why not have some fun and #tweetapaper?
