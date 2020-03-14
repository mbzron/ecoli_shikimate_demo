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
m = project.models[1]  # load E. coli GSM iJO1366 
```

The next step is to check that the model supports growth on glucose and save the corresponding constraints as a 
'conditions' .json file,

```python
sol = m.optimize()  # maximise growth on glucose
print('growth rate (glc):',sol.objective_value)  # print result
project.save_conditions(m, 'glc_growth')  # create conditions file 
```