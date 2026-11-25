# Homophily Analysis and Social-Aware Music Discovery on Last.fm

**Term project proposal · BLM4150 Introduction to Social Network Analysis**  
Department of Computer Engineering, Bandırma Onyedi Eylül University · Fall 2026–2027

## Research questions

Do Last.fm friends listen to similar artists after accounting for the popularity of those artists? Shared interest in a chart-topping artist provides weaker evidence of an association between friendship and taste than shared interest in an artist with few listeners. We will estimate this association, then investigate whether a user's known friendships help recommend artists when their own listening history is unavailable. We will also test whether musical similarity helps recover hidden friendships and identify artists whose audiences span several friendship communities.

## Data and network design

We use the **HetRec 2011 Last.fm 2K** research dataset. The Information Retrieval Group at Universidad Autónoma de Madrid generated the dataset from Last.fm, and GroupLens distributes the release. The archive contains tab-separated records of friendships, artist play counts, and tags assigned to artists by users. The release permits noncommercial use. [GroupLens dataset and release documentation](https://grouplens.org/datasets/hetrec-2011/) · [Last.fm](https://www.last.fm/)

| Layer | Nodes | Relationship | Observed size |
|---|---|---|---:|
| Friendship | Users | Undirected, unweighted friendship | 12,717 unique pairs |
| Listening | Users and artists | Directed user-to-artist edge weighted by play count | 92,834 unique pairs |

The two layers contain 1,892 users and 17,632 artists, giving 19,524 nodes and 105,551 edges in total. We distinguish user IDs from artist IDs even when their numerical values coincide. The friendship file lists both directions of each friendship. Counting each unordered pair once gives the first row of the table. A listening edge aggregates plays for one user and one artist. For example, the edge from user 2 to Duran Duran has weight 13,883. User 275 is a friend of user 2 and has 77 recorded plays of the same artist.

Tags will provide features for the artists they cover. Of the 17,632 artists in the listening network, 12,133 have a matching tag assignment. We will keep the full listening network, record where tag similarity is available, and report coverage for analyses that use tags.

## Analysis plan

**Network structure.** On the friendship layer we will measure degree distribution, density, connected components, clustering, and degree assortativity. For global path length and diameter, we will use the largest connected component and report its size. We will compare user rankings by degree, betweenness, closeness, and PageRank. On the listening layer we will distinguish the breadth of an artist's audience (unique listeners) from the volume of listening (total plays).

**Musical homophily.** For each user pair we will compare Jaccard similarity of artist sets, cosine similarity of play-count vectors, and cosine similarity of user profiles aggregated from artist tag TF-IDF vectors. Our descriptive comparison across all pairs yielded mean artist-set Jaccard similarities of **0.1026 for 12,717 friend pairs** and **0.0236 for 1,776,169 pairs without a recorded friendship**. These estimates precede adjustment for popularity. We propose a permutation test that preserves each listener's number of artists and each artist's listener count while randomizing the listening links. The difference in mean Jaccard similarity will be the test statistic. A Mann–Whitney U rank comparison will supplement the description, with uncertainty assessed through permutation because multiple pairs share the same users. We will also examine whether cosine similarity between users' Node2Vec embeddings agrees with similarity in their listening profiles.

**Communities and bridge artists.** We will detect user communities with Louvain and report modularity. For each artist we will compute the share of distinct listeners in each community and the entropy of these shares. To account for popularity and community size, we will compare the observed entropy with the distribution obtained by sampling the same number of listeners from the network. We will report listener counts and community counts alongside the bridge rankings and examine how rankings change with a minimum audience size.

## Recommendation experiments

| Task | Proposed comparison | Evaluation |
|---|---|---|
| Recommend artists to a user with known friends and hidden listening history | Global artist popularity, direct friends' listening, and a combined ranking using friends and users with similar friendship-network Node2Vec embeddings | Mean per-user Precision@K and Recall@K against the hidden artist sets |
| Recover a hidden friendship | A gradient boosting classifier using music similarity, common neighbors, Adamic–Adar, neighbor-set Jaccard, preferential attachment, and Node2Vec similarity | ROC AUC and average precision, reported separately for random unlinked pairs and unlinked pairs sharing a training-graph neighbor |

For the first task we will hold out entire listening profiles. Artist popularity and listening-based scores will use training users only. We will report the chosen cutoff, the candidate artist set, and how many held-out users have friends with available training profiles. For the second task we will partition undirected friendships before computing structural features or fitting embeddings. We will tune on validation data and compare four feature sets on the same test pairs: music only, structure only, Node2Vec only, and all features combined. Here, structure means the explicitly calculated neighborhood features. Negative sampling will exclude every recorded positive pair.

## Interpretation and reproducibility

Profiles contain at most 50 recorded artists per user, so the analysis describes overlap in the available listening profiles. The friendship data provide a snapshot without link-formation dates. This design can estimate an association between friendship and taste and test recovery of hidden links. Distinguishing friend selection from social influence would require observations of how friendships and listening changed over time.

We will use Python in Jupyter, with pandas and NumPy for data preparation, NetworkX for graph analysis, SciPy for statistical tests, Node2Vec for embeddings, scikit-learn for modeling and evaluation, and Matplotlib for network figures. The analysis will document data checks, split definitions, random seeds, negative sampling ratios, and metric conventions alongside the code. Predictive performance and popularity-adjusted homophily remain open empirical questions.
