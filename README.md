# GSModutils demo
## Shikimate production in *E. coli*
Demonstration of metabolic model management with [gsmodutils](https://academic.oup.com/bioinformatics/article/35/18/3397/5317162).

Based on shikimate production in *E. coli* as described by [Fujiwara et al. 2020](https://www.nature.com/articles/s41467-019-14024-1).

Start by opening a python console and importing GSMProject and cobrapy,

```python
from gsmodutils import GSMProject
import cobra
```

then load the current project,

```python
project = GSMProject('.')  # load project from current directory
```

and load the *E. coli* [genome scale model](https://www.embopress.org/doi/full/10.1038/msb.2011.65).

```python
m = project.models  # load E. coli GSM iJO1366 
```

The next step is to define conditions under which *E. coli* strains will be grown

```python
m.exchanges.get_by_id('EX_xyl__D_e').bounds = (-10,1000)  # allow xylose uptake
m.reactions.get_by_id('EX_tyr__L_e').bounds = (-10,1000)  # allow tyrosine uptake
m.reactions.get_by_id('EX_phe__L_e').bounds = (-10,1000)  # allow phenylalanine uptake
m.reactions.get_by_id('EX_o2_e').bounds = (-0.1,0)  # microaerobic conditions
m.reactions.get_by_id('EX_co2_e').bounds = (0,1000)  # co2 product only
```

and save the corresponding constraints as a 'conditions' .json file,

```python
project.save_conditions(model=m, conditions_id='glc+xyl_M9_microaero')
```

then repeat this step, creating conditions for growth on xylose, and glucose only.
Also create a conditions file for aerobic growth by altering 
the bounds of the oxygen boundary reaction ('EX_o2_e').

At this stage, the gsmodutils test utility can be executed from the command line

```shell script
$ gsmodutils test
```

which shows that the model supports growth on the expected substrates

```shell script
    --model::iJO1366.json::conditions::glc+xyl_M9_aero

    --model::iJO1366.json::conditions::glc+xyl_M9_microaero

    ...
```

These basic tests can be run at any time to monitor model behaviour after modification.

*NB: NOT SURE IF PENTOSE PHOSPHATE STUFF STILL RELEVANT*

One aim stated in Fujiwara *et al.* is to create a "xylose catabolic pathway that directly flows into the 
TCA cycle without interfering glycolysis and PPP."
The diagram below is a general (and simplified) scheme of the pentose phosphate pathway, 

![pentose phosphate pathway](./figures/pentose_phosphate_shadow.png)

where 1 = glucose phosphorylation to glucose-6p; 2 = xylulose phosphorylation to xylulose-5p; 3 = transketolase 1, 
phosphofructokinase, fructose-bisphosphate aldolase, topoisomerase; 4 = ribose-phosphate epimerase, ribulose-phosphate 
epimerase, ...; 5 = isomerase; 6 = fructose-6p phosphorylation and cleavage; 7 = transketolase 2; 8 = ...; 
and 9 = further glycolysis.

We want to monitor the activity of this pathway, so an 'essential pathway' test is made.

To write an essential pathway test, store reaction flux ranges in a dictionary, and write a test file using

```python
project.add_essential_pathway(...)
```

specifying a test name, description, models, conditions, and reaction_fluxes.
This essential pathway test can be run from the command line at any time with 'gsmodutils test'.

The metabolic engineering strategy conducted by Fujiwara et al. involves the creation of
three mutants: CTF5, G1 and G1x.

CTF5 contains disruption of genes encoding Pyruvate Kinase (PK)
and the Phosphotransferase System (PTS). In addition, pathways towards amino acids are disrupted so that the organism 
becomes auxotrophic for tyrosin and phenylalanine. These gene disruptions (termed 'knock-outs')
are carried out follows, using 'save_design' to save the mutant as a GSModutils design.

```python
aa_pathway_genes = list(m.reactions.PPNDH.genes)+list(m.reactions.PPND.genes)  # amino acid pathway genes (phe/tyr)
pts_system_genes = list(m.reactions.GLCptspp.genes)  # genes encoding pts transport system genes

gene_ko_list = aa_pathway_genes + pts_system_genes + list(m.reactions.PYK.genes)

cobra.manipulation.delete_model_genes(m,gene_ko_list)

# save ctf5 design
project.save_design(m,
                    'CTF5',
                    'CTF5 base strain',
                    conditions='glc+xyl_M9_microaero',
                    base_model='iJO1366.json'
                   )

# save ctf5 strain with malate in the 'medium'
project.save_design(m,
                    'CTF5mal',
                    'CTF5 base strain with malate',
                    conditions='glc+xyl+mal_M9_microaero',
                    base_model='iJO1366.json'
                   )
```

Designs are also created for G1, which requires knockouts of genes encoding 
reactions with IDs EDA, PPS, PPC and PPCK; and G1x, 
which requires the addition of the Dahms pathway...

*FIGURE AT THIS POINT?*

Test are written to monitor the activity of the Dahms pathway and TCA cycle, which should 
function independently based on the provision of xylose and/or glucose.

Finally, a custom test is written to looks at the shikimate producing behaviour of the 
different strains under different conditions.