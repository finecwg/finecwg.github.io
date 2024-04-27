---
layout: single
title: "Gut microbiome-mediated epigenetic regulation of brain disorder and application of machine learning for multi-omics data analysis"
categories: reviews-basic-biomedical-science
tags: [paper_reviews, brain, neuroscience, gut microbiome]
toc: true
toc_sticky: true
toc_label: "microbiome-mediated epigenetic regulation in brain"
author_profile: false
---

_this is the re-uploaded version of my previous [Naver Blog Posting](https://blog.naver.com/finecwg/222799818594) at 2022-07-04_

Cite: Kaur, H., Singh, Y., Singh, S., & Singh, R. B. (2021). Gut microbiome-mediated epigenetic regulation of brain disorder and application of machine learning for multi-omics data analysis. Genome
, 64
(4), 355-371.

[Paper Link](https://cdnsciencepub.com/doi/full/10.1139/gen-2020-0136)

# Abrstract

- GBA(gut-brain axis)
  - connect CNS ↔ ENS(enteric nervous system)
- gut microbiota: key regulator of GBA
  - modulate CNS through neuroendocrine and metabolic pathways
  - depression, anxiety, autism, stroke, pathophysiology of other neurodegenerative diseases
  - microbe-derived metabolites: can influence host metabolism by acting as epigenetic regulators
    - ex> butyrate(intestinal bacterial metabolite): histone deacetylase inhibitor → improve learning and memory in animal model
- multi-omics approach that utilizes AI and ML to analyze/integrate omics data is necessary
  to better understand the role fo the GBA in pathogenesis of neurological disorders
- current understanding of epigenetic regulation of the GBA and proposes a framework to integrate multi-omics data for prediction, prevention, and development of precision health approaches to treat brain disorders

# Introduction

CNS↔ENS biochemical link: GBA

ENS: ‘second brain’

- more than 30 neurotransmitters(most identical to CNS): acetylcholine, dopamine, serotonin
- CNS, ENS, ANS(autonomic)
- complex peripheral neural circuit embeded within its wall comprising sensory neurons, motor neurons, and interneurons
- 500 million neurons (5 times larger than spinal cord)
- independently regulate basic gastrointenstinal functions
- brain→ENS indirectly(change in gastrointenstinal motrility, secretion, intenstinal permeability) or directly(signaling molecules)
- gut→brain: neural, endocrine, immune, humoral pathways
- psychiatric disorders(anxiety, depression), neurological, neurodevelopmental disorders(autism, AD, PD..)
- gut microbiota: epigenetic modifications(DNA methylation, histone modifications) → influence brain and behavior

![**Fig 1.** 
Bidirectional communication between the brain and gut mediated via the gut microbiota: black
The effect of gut microbiome on brain functions via epigenetic regulation of gene expression: blue](/images/2024-04-27-microbiome-epigenetics/Untitled.png)

**Fig 1.**
Bidirectional communication between the brain and gut mediated via the gut microbiota: black
The effect of gut microbiome on brain functions via epigenetic regulation of gene expression: blue

# Gut Microbiome

key regulator of the GBA

total of all microorganisms in gastrointestinal tract

bacteria, fungi, archaea, viruses, protozoans

$10^14$ microbes

symbiotic relationship with gut mucosa ⇒ impart metabolic, immunological, gut protective functions in the host

**microbiome**: collective genomes of the microorganisms(i.e., microorganism+their set of genes)

- firmicutes, bacteroidetes: 90%
- actinobacteria, proteobacteria: 1~5%

in human diseases…

inflammatory bowel diseases

irritable bowel syndrome

metabolic diseases(obesity, diabetes), allergic diseases, neurodevelopmental illnesses

important in human development, immunity, nutrition

2007 NIH: Human Microbiome Project

provided vast knowledge: dvelopment of reference seq. multi-omics datasets, computational/statistical tools, analytical/clinical protocols → broader research community

# Gut microbiota-brain communication (via vagus nerve, neuroendocrine, and neuroimmune pathways)

- direct signals to the brain by activating afferent sensory neurons of the vagus nerve
  : via neuroimmune and neuroendocrine pathways

1. **vagus nerve**

10th cranial nerve(from brainstem-neck-thorax down to the abdomen)

main component of the parasympathetic NS

crucial bodily functions: control of mood, immune response, digestion, heart rate

modulate gastrointestinal motility, secretion, epithelial permeability

⇒ modify the physical environment that the microbiota inhabit, thus affecting its composition

transmit signals from the gastrointestinal tract to the CNS and can recognize microbioal products or cell wall components

- stimulation of vagus nerve
  stabilize the abnormal electrical activity in the brain ⇒ treatment of epilepsy and other neurological conditions

ablated gut-related vagal communication → affect adult neurogenesis, stress reactivity, anxiety-and fear-related behavior, cognition (psychiatric disease)

- Gut bacteria’s influence
  influence the cells of the gastrointestinal tract that produce neurotransmitters and digestive hormones or peptides in the gut ⇒ can alter the brain and behavior
  diet, environment, probiotics, drugs(antibiotics etc.): affect vagus nerve activity
  vagal afferent nerve fibers respond to a variety of stimuli(cytokines, nutrients, gut peptides, hormones)
  vagal receptors: recognize the regulatory gut peptides, inflammatory molecules, dietary components, bacterial metabolites to relay signals to the CNS
  altered gut microbiota composition/changes ⇒ vagus nerve to alter CNS outputs(active role in mediating neurological functions)
  this nerve: direct route for gut-to-brain signaling
- _Lactobacillus rhamnosus_ treatment in mice ⇒ reduced stress-induced corticosterone and anxiety- and depression-related behavior(not in vagotomized mice)
  ⇒vagus: major modulatory constitutive communication pathway bet. bacteria and brain
- _Lactobacillus intestinalis_, _Lactobacillus reuteri_ cause depression, anhedonia-like phenotypes in antibiotic-treated mice via the vagus nerve
- _L.reuteri_: target ion channels in enteric sensory neurons ⇒ increased action potential

1. **Neuroendocrine system**

- potential link bet. gut microbiota and neuroendocrine system in various psychiatric, gastrointestinal diseases

1. **Immunomodulation by the microbiota**

- orchestrates microbiota-gut-brain communication
- gut microbiota regulate inflammation, autoimmunity, immune cell trafficking
- contribute to microglial maturation and function
  - microglial involvement existing neurological disorders (human) -> altered microbial community composition was observed
- active microbial signaling is required throughout adulthood to preserve microglial maturation

# Gut metabolites (short-chain fatty acids)

produce wide range of metabolites

neurotransmitters, hormones, vitamins, short-chain fatty acids(SCFAs)

play an important role in host defense(training host immune system)

affect neuronal functions

- commensal bacteria(anaerobic): fermentation of non-digestible dietary fibers
  ⇒produce key bacterial metabolites and SCFAs (acetate, propionate, butyrate, valerate)
  ⇒regulate gut integrity and immune responses and maintain overall host homeostatis
  More than 95% of these SCFAs are used as an energy source by colonocytes, and 60-70% of their energy comes from SCFA oxidation.
  Role of SCFAs
  - act as signaling molecules (involved in systemic lipid metabolism and glucose/insulin regulation)
  - cross BBB via monocarboxylate transporters
  - essential for maintaining the intestinal barrier permeability
    : regulate tight junction proteins in the gut
  - involve in maintaining BBB permeability
  - modulate mammalian cell functions by serving as an energy substrate or by signaling through GPCRs(GPR41, GPR43)
  - at Brain, influence CNS immune system (regulating microglial maturation/function)
    -dysfunction of microglia is involved in the initiation or progression of multiple CNS disease(AD, PD, ASD, depression)

# Role of gut microbiota in brain disorders

microbiota dysbiosis in depression, anxiety, autism, neurodegenerative, stroke

can be used as biomarker for the CNS

- brain alters intestinal microenvironment by regulating the gut motility and secretion as well as mucosal immunity via the neuronal glial-epithelial axis and visceral nerves
- gut reacts via changing the bacterial metabolites (neurotransmitters, neuromodulators like SCFAs, gut hormones(leptin, ghrelin etc.))

probiotics/antibiotics usage → behavioral responses

anxiety-like disorders & improvement of anxiety symptoms by regulating intestinal microbiota

Cerebral disorders → various psychiatric symptoms (dementia, mood disorder, stress, anxiety, movement disorder, personality disorder…)

talk therapy + antidepressant medication treatment

antidepressent - directly regulate HPA axis

HPA axis → contribute

Gut microbiota

- development of the HPA axis
- GBA disruption → change intestinal motility/secretion/visceral hypersensitivity → cellular alterations of the entero-endocrine and immune systems

1. ASD

   environmental factors

   immune system abnormalities

   gut microbiota

2. AD

   altered gut microbial composition → associated with increased intestinal permeability and increased intestinal inflammation in various models of AD

# Gut metabolites as epigenetic regulators

- cell number: 10 times than the totl number of human body cells
- 3~4 million unique genes present (100-150 times more genes in geome)

metabolites: some are absorbed into the systemic circulation and reach distant organs, including brain

some are biologically active, some are further metabolized by host enzymes

- important epigenetic regulators: can influence host gene expression
  : interact at DNA, RNA, histone levels

DNA methylation, post-transcriptional histone modifications, chromatin restructuring and associated DNA methyltransferases(DNMTs), DNa hydroxylases, histone acetyltransferases, histone deacetylases, histone methyltransferases, histone demethylases

- DNA methylation
  CpG regions(C-G rich regions): by DNMTs → suppress gene transcription
  diet, neutrients, environmental agents(toxic, pharmacologiclal factors)
  no microbiota or no change → long-lasting epigenetic modifications(affect behavior)
  - TET: DNA demethylation
- _Clostridia_ 등
  dietary fibers를 SCFAs로, 특히 butyrate: HDAC inhibitor
  ⇒enhance chromatin accessibility, activate gene expression
- histone acetylation
- histone methylation
  - H3 lysine 4: gene activity 증가
  - lysine 9: repression
- gut microbiota: alterate chromatin state
  - acetate, propionate, folate 등
  - acetate, propionate: HDAC inhibitory activity
  - SAM(S-adenosyl methionine) (MAT가 methione에서 이거되는거 촉매)
    ⇒DNMT가 methyl group을 CpG로 운반 → ㅎㄷㅜㄷ 냐ㅣ두챠ㅜㅎ
  - 5mC(5-methylcytosine, fifth base of DNA)…

![Untitled](/images//Untitled 1.png)

- vagus nerve in epigenetic regulation
  - vagus nerve stiulation(VNS) → hippocampal, cortical, blood epigenetic transcriptomes, epigenetically modulates neuronal plasticity, stress-response signaling genes
  - glucose affects a cell-specific epigenetic regulation of memory-related genes(_Bdnf, Fgf-1_)
  - glucose-mediated secretion of gut hormones may induce VNS to enhance memory process
  - gut microbiota!!
- how gut microbiota influence the epigenetics in brain disorders?(especially neuropsychiatric disorders: anxiety, depression, neurodegenerative)
  - bile acids, vitamins, choline metabolites, SCFAs/lipids
  - SCFAs: regulate MHC gene expression
  - butyrate: HDAC inhibitor

# Application of machine learning in multi-omics data analysis

![**Fig 2.** Framework for multi-omics data integration and use of advanced ML approaches to predict the health or diseased state or to find the right therapeutic strategy for an individual person](/images//Untitled 2.png)

**Fig 2.** Framework for multi-omics data integration and use of advanced ML approaches to predict the health or diseased state or to find the right therapeutic strategy for an individual person

# Discussion

- GBA: biochemical link bet. CNS and ENS ← gut microbiota is the main contributor to variable functions
- gut microbiome ↔ brain function : critical in understanding role of GBA
- gut microbiota → brain
  - through neural, endocrine, immune, humoral pathways
  - microbiology, epigenetics, computational biology
- metabolites: SCFAs
  - host defense, act as signaling molecules involved in lipid metabolism, glucose/insulin regulation
  - essential for maintaining intestinal, BBB permeability
  - modulate mammalian cell functions by serving as an energy substrate / signaling through GPCR
  - (in)directly stimulate vagal nerve → influencing GBA
- epigenetic
  - diet, nutrients, environments, probiotics, drugs…
  - SCFAs: HDAC inhibitor etc…
- omics/computational biology
  - identification of biological factors/mechanisms that connect the onset and progression of neurological diseases to gut microbiome
  - disease risk prediction and precision medicine in humans
  - but not yet
