## Retrieve old and new revisions of an introduction from a Wikipedia article.

```python
from wiki_data_fetcher import *

# Option 1: Get a revision from n days ago
title = "Albert Einstein"
new_info = get_revision_from_age(title, age_days = 0)   # current revision
old_info = get_revision_from_age(title, age_days = 10)  # ten days ago
# Option 2: Get the nth revision before current
json_data = get_previous_revisions(title, revisions = 100)
old_info = extract_revision_info(json_data, 100)

# new_info and old_info are dictionaries:
# {'revid': 1143737878, 'timestamp': '2023-03-09T15:49:20Z'}
# Now get the introduction (the text before the first <h2> heading) for each revision
new_revision = get_wikipedia_introduction(new_info["revid"])
old_revision = get_wikipedia_introduction(old_info["revid"])
```

## Get the number of revisions back for a given revid
```python
title = "Albert Einstein"

# The current revision
json_data = get_previous_revisions(title, revisions = 0)
current_info = extract_revision_info(json_data, 0)
get_revisions_behind(title, current_info["revid"])  # should be 0

# A previous revision
old_info = get_revision_from_age(title, age_days = 100)
get_revisions_behind(title, old_info["revid"])

# A very old revision
very_old_info = get_revision_from_age(title, age_days = 5000)
# This returns -1000 because we only search back 1000 revisions
get_revisions_behind(title, very_old_info["revid"])
```

## Classify the differences between the revisions as noteworthy or not, and provide a rationale.

```python
from models import *
classifier(old_revision, new_revision, "heuristic")
```

```
{'noteworthy': True,
 'rationale': 'The differences are noteworthy because the new revision adds the specific outcome of Einstein\'s recommendation (the Manhattan Project), clarifies his famous objection to quantum theory with a direct quote ("God does not play dice"), and provides the full rationale for his Nobel Prize, all of which add significant details about major events and his views.'}
```

