# QuestEval

QuestEval is a quest for an **NLG metric** to assess if two different inputs contain the same information. The metric, based on Question Generation and Answering can deal with **multimodal** and **multilingual** inputs. 
It is the result from an (on-going) international collaboration, and so far it tackles various tasks:

- [Summarization](#summarization)
- [Text Simplification](#text-simplification)
- [Data2text](#data2text)
- [Multilingual Evaluation](#multilingual-evaluation)

Planned extensions: 
- machine translation
- image captioning 
- multilingual summarization

## Installing QuestEval
```
$ conda create --name questeval python=3.7
$ conda activate questeval
```
**WARNING**: You need to install, before any package, correct version of [pytorch](https://pytorch.org/get-started/locally/#start-locally) linked to your cuda version.
```
(questeval) $ conda install pytorch cudatoolkit=10.1 -c pytorch
```

```
(questeval) $ conda install pip
(questeval) $ pip install -e .
```

todo
Now, download the trained [models](https://drive.google.com/file/d/1CnFImB39vIYAxYtqNecPk3s1jjv1o3zr/view?usp=sharing) and unzip the file *models* file in the current root folder *safeval*.
Those QA/QG/Policy models are the one used to compute all the scores reported in the paper. Alternatively, you can use your own models.

## Using QuestEval 

The default task is *text2text* and the default language *en*, allowing to measure the similarity of content between any English texts:

```
from questeval.questeval_metric import QuestEval

summary = """After wildfires consumed an entire town, students and teachers who had planned for remote classes found some comfort in staying connected amid the chaos."""
article = """Ash fell from an apocalyptic orange sky as Jennifer Willin drove home last week from the only school in tiny Berry Creek, Calif., where she had picked up a pair of Wi-Fi hot spots for her daughters’ remote classes. Hours later, her cellphone erupted with an emergency alert: Evacuate immediately. By the next morning, what one official described as a “massive wall of fire” had swept through the entire Northern California town of about 1,200 people, killing nine residents, including a 16-year-old boy, and destroying the school and almost every home and business. Ms. Willin and her family escaped to a cramped hotel room 60 miles away. In her panic, she had forgotten to grab masks, but she had the hot spots, along with her daughters’ laptops and school books. On Monday, the two girls plan to meet with their teachers on Zoom, seeking some comfort amid the chaos. They’re still able to be in school,” Ms. Willin said, “even though the school burned to the ground.” As the worst wildfire season in decades scorches the West amid a still raging pandemic, families and educators who were already starting the strangest and most challenging school year of their lifetimes have been traumatized all over again. Tens of thousands of people have been forced to flee their homes, with some mourning the loss of their entire communities. But amid the twin disasters, the remote learning preparations that schools made for the coronavirus are providing a strange modicum of stability for teachers and students, letting many stay connected and take comfort in an unexpected form of virtual community."""

questeval = QuestEval(isCuda=True)
score = questeval.compute_all(summary, article)
print(score['scores'])
```
Output:
```
{'fscore': 0.2883088133952934,
 'precision': 0.5038477301470266,
 'recall': 0.07276989664356022}
```

Yes, it works without any references, but if you have it you can also pass it as input:
```
reference = """After wildfires consumed an entire town, students and teachers who had planned for remote classes found some comfort in staying connected amid the chaos."""
score = questeval.compute_all(summary, article, reference)
print(score['scores'])
```
Output:
```
{'fscore': 0.48813298610456035,
 'precision': 0.5959027691779738,
 'recall': 0.38036320303114696}
```
Yes, if the reference is similar to the evaluated text, the score improved which is expected! Note that the score is between 0 and 1.

In addition, you can access all the logs including the generated questions and predicted answers. For instance, the generated questions on the source that were asked on the hypothesis are available via:
```
print(score['logs']['src_hyp']['questions'])
```


For tasks specificities, see bellow. In particular for:
- [Summarization](#summarization)
- [Text Simplification](#text-simplification)
- [Data2text](#data2text)
- [Multilingual Evaluation](#multilingual-evaluation)

We also provide more examples in the Jupyter notebook *example/Examples.ipynb*. The notebook also contains all the code to reproduce the results in the paper *SAFEval: Summarization Asks for Fact-based Evaluation*.

For using a virtual jupyter environment:

```
(questeval) $ conda install jupyter
(questeval) $ python -m ipykernel install --user --name=questeval
(questeval) $ pip install matplotlib
(questeval) $ pip install ipywidgets
```

## Tasks specificities:

### Summarization:
The project is a collaboration work between [LIP6](https://mlia.lip6.fr/) Lab and [ReciTAL Research](https://recital.ai/en/research-development/).

Questeval also handle specificities of summarization with a Weighter to select only the questions asking about important facts that are worth to be summarized: read more in the original paper [TODO]. To activate this Weighter: *do_weighter=True*.

Paper link: [todo]

How to cite:
```
@article{scialom2020SAFEval,
  title={SAFEval: Summarization Asks for Fact-based Evaluation},
  author={Scialom, Thomas R and Dray, Paul-Alexis and Lamprier Sylvain and Piwowarski Benjamin and Staiano Jacopo},
  journal={arXiv preprint arXiv:2007.12626 [todo]},
  year={2021}
}
```

### Text Simplification:

The metric is effective with its default parameters. It ranks better the systems than BLEU or SARI metrics as reported in [TODO].


### Data2text:

We propose by default trained QA/QG models dealing with table inputs ((e.g. E2E or Webnlg, see more [todo]). To load QuestEval for data2text tasks, specify *task=e2e* or *task=webnlg*. Note that you need a specific processing to linearised the tables. By default we handle the [GEM](https://gem-benchmark.com/) format for these two datasets. If you need an other preprocessing of the table, you can pass your custom function to Questeval: *src_preproc_pipe=custom_formating*.

### Multilingual Evaluation:

QuestEval support non english evaluation: the parameter **language* can be set to *multi* to tackle multilingual data. For Question Generation and Answering we used the m-Minilm model [TODO]. For the answer selection, spacy does not support multilingual *noun chuncking* which might lead to poorer correlation with human jugement than in English Questeval: we are working on that!

