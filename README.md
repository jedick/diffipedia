# Noteworthy Differences

This is an AI alignment project.

- The data are old and new revisions of Wikipedia articles
- Two prompts (heuristic and few-shot) are used to classify differences between the revisions as noteworthy or not
- Examples where the prompts agree are removed from the alignment pipeline
  - This allows the human judge to focus on the hardest examples
- Examples where the prompts disagree are pipelined for human and AI judges
- The AI judge is run in two modes: unaligned (minimal instructions) and aligned
  - The alignment instructions are based on the human judge's rationales.
- The performance between the unaligned and aligned AI judges is measured on unseen data to test the alignment.

## Interactive usage

Retrieve old and new revisions of an introduction from a Wikipedia article.

```python
from wiki_data_fetcher import *

# Option 1: Get a revision from n days ago
title = "Albert Einstein"
new_info = get_revision_from_age(title, age_days = 0)
old_info = get_revision_from_age(title, age_days = 10)
# Option 2: Get the nth revision before current
json_data = get_previous_revisions(title, revisions = 100)
old_info = extract_revision_info(json_data, 100)

# new_info and old_info are dictionaries:
# {'revid': 1143737878, 'timestamp': '2023-03-09T15:49:20Z'}
# Now get the introduction (the text before the first <h2> heading) for each revision
new_revision = get_wikipedia_introduction(title, new_info["revid"])
old_revision = get_wikipedia_introduction(title, old_info["revid"])
```

Classify the differences between the revisions as noteworthy or not, and provide a rationale.

```python
from models import *
classify(old_revision, new_revision, "heuristic")
```

```
{'noteworthy': True,
 'rationale': 'The differences are noteworthy because the new revision adds the specific outcome of Einstein\'s recommendation (the Manhattan Project), clarifies his famous objection to quantum theory with a direct quote ("God does not play dice"), and provides the full rationale for his Nobel Prize, all of which add significant details about major events and his views.'}
```

## AI alignment: overview

There are two classifier models with different prompts: heuristic and few-shot.
The judge only comes in when the classifiers disagree.
The purpose of alignment is to get the AI judge to make more human-like decisions on hard examples.
For this reason, the prompts for the classifiers (heuristic and few-shot) are written once and locked in for the duration of the alignment.

1. Collect data
    - Introductions from Wikipedia articles: old and new revisions
2. Create examples
    - Classify differences using two prompt styles: heuristic and few-shot
3. Human judge
    - Judge examples where heuristic and few-shot classifiers disagree
    - No AI context: Judge is blind to classifiers' output
4. AI judge (pre-alignment)
    - Judge examples where heuristic and few-shot disagree
    - AI context: Judge can see classifiers' output
5. Alignment
    - Add alignment text to prompt for AI judge
    - The text consists of three rationales (from the few-shot and heuristic prompts and the human judge) and the classification from the human judge
    - The competing rationales provide context that may help with reasoning and generalization
6. Evaluate
    - Measure performance change (accuracy with human judge as ground truth) between unaligned and aligned AI judges
7. Test
    - Repeat steps 1-4 and 6 to measure the performance change on unseen data
8. Iterate
    - Repeat alignment (step 5) until acceptable performance on test data is reached

## AI alignment: instructions
 
**Initial preparation:** Run `data/get_titles.R` to extract and save the page titles linked from the Wikipedia Main Page to `data/wikipedia_titles.txt`.
*This is optional; do this to use a newer set of page titles than the ones provided here.*
  
1. **Collect data:** Run `collect_data.py` to retrieve revision id, timestamp, and page introductions for 0, 10, and 100 revisions before current.
The results are saved to `data/wikipedia_introductions.csv`.

2. **Create examples:** Run `create_examples.py` to run the classifier and save the results to `data/examples.csv`.
The model is run up to four times for each example:
two prompt styles (heuristic and few-shot) and two revision intervals (from 10th and 100th previous revisions to current).

3. **Human judge:** Run `data/extract_disagreements.R` to extract the examples where the heuristic and few-shot classifiers disagree.
These are saved in `data/disagreements_for_human.csv` (only Wikipedia introductions) and `data/disagreements_for_AI.csv` (introductions and classifier responses).
*Without looking at the classifier responses*,
the human judge fills in the `noteworthy` (True/False) and `rationale` columns in the for-human CSV file and saves it as `data/human_judgments.csv`.

4. **AI judge:** Run `judge_disagreements.py` to run the unaligned judge on the examples where the classifiers disagree.
The AI judge only sees the old and new revisions and the rationales from the heuristic and few-shot classifiers (not the human judge's responses).
The results are saved to `data/AI_judgments.csv`.

5. **Alignment:** Run `data/align_judge.R` to collect the alignment data
(rationales and classification from human judge, and rationales from heuristic and few-shot prompts) into `data/alignment_text.txt`

6. **Evaluate:** Run `judge_disagreements.py --aligned` to run the aligned judge on the same examples where the classifiers disagree,
then run `data/summarize_results.R` to compute the summary statistics (results listed below).

## Results

- Wikipedia pages processed: 95
- Available 10th previous revision: 94; 100th previous revision: 81
- Revisions classified as noteworthy with heuristic prompt: 29%; few-shot prompt: 35%
- Disagreements between heuristic and few-shot prompts: 17
  - Classified as noteworthy with heuristic prompt: 3; few-shot prompt: 14
  - Classified as noteworthy by human judge: 11
  - Classified as noteworthy by **unaligned** AI judge: 17 (65% accurate)
  - Classified as noteworthy by **aligned** AI judge: 12 (94% accurate)

