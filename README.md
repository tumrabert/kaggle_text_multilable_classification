# Research Paper Multilable Classification Project

This repository contains code for a Kaggle competition on classifying research papers into multiple subject areas using both traditional and deep learning approaches.

# Chapter 1 : **Introduction**

## Project Purpose

Classified research papers into 18 classes subject areas including

- CE - Civil Engineering
- ENV- Environmental Engineering
- BME - Biomedical Engineering
- PE - Petroleum Engineering
- METAL- Metallurgical Engineering
- ME - Mechanical Engineering
- EE - Electrical Engineering
- CPE - Computer Engineering
- OPTIC - Optical Engineering
- NANO - Nano Engineering
- CHE - Chemical Engineering
- MATENG - Materials Engineering
- AGRI - Agricultural Engineering
- EDU - Education
- IE - Industrial Engineering
- SAFETY - Safety Engineering
- MATH - Mathematics and Statistics
- MATSCI - Material Science

This type of classification supports researchers in finding relevant literature, improves search results in academic databases, and helps to organize collections of scientific publications.

## Multi-label Classification

I use multi-label classification because a research paper is belong to multiple subject areas. For example, a paper might intersect Biomedical Engineering and Nano technology.

## Project Structure

This report delves into data preparation, model development, results analysis, discussion of potential improvements, and conclusions.

---

# Chapter 2: **Data preparation**

## Data Source:

This dataset contains 454 research paper samples, including their titles, abstracts, and class labels. The data is provided in JSON format, split into two files:

- **train_for_student.json**: The training dataset used to build machine learning models.
- **test_for_student.json**: The testing dataset used to evaluate the performance of trained models.

**JSON Structure**

Each JSON files has the following structure:

- Top-Level Object: The entire dataset is represented as a single JSON object.
- Individual Records: Within the top-level object, each research paper sample is represented as a key-value entry.
  - Key: A unique identifier for the sample (e.g., "001", "002eval").
  - Value: An object containing the following information about the research paper:
    - Title: The title of the research paper.
    - Abstract: Abstarct of the research paper.
    - Classes: A list of classes/categories the research paper belongs to (e.g., "CHE", "MATENG", "CPE").

**train_for_student.json**

```json TRAIN
{
    "001": {
        "Title": "Activated carbon derived from bacterial cellulose and its use as catalyst support for ethanol conversion to ethylene",
        "Abstract": "\u00a9 2019 Elsevier B.V.Activated carbon derived from bacterial cellulose (BC-AC) ...",
        "Classes": [
            "CHE",
            "MATENG"
        ]
    },
    "002": {
        "Title": "The algorithm of static hand gesture recognition using rule-based classification",
        "Abstract": "\u00a9 Springer International Publishing AG 2018.Technology becomes a part of ...",
        "Classes": [
            "CPE"
        ]
    },
    ...
}
```

**test_for_student.json**

```json TEST
{
    "001eval": {
        "Title": "Comparative Electrical Energy Yield Performance of Micro-Inverter PV Systems Using a Machine Learning Approach Based on a Mixed-Effect Model of Real Datasets",
        "Abstract": "\u00a9 2013 IEEE.Long-term energy evaluation of PV systems that use micro-inverter ..."
    },
    "002eval": {
        "Title": "Effects of graphene nanoplatelets on bio-based shape memory polymers from benzoxazine/epoxy copolymers actuated by near-infrared light",
        "Abstract": "\u00a9 The Author(s) 2021.Novel near-infrared (NIR) light-induced bio-based shape ..."
    },
    ...

}
```

## Preprocessing Steps:

- Turn target classes into a binary matrix representation where each column represents a possible class

  ```python
  multilabel = MultiLabelBinarizer()
  y = multilabel.fit_transform(df['Classes'])
  ```

- Feature Engineering:

  Used TF-IDF (Term Frequency-Inverse Document Frequency) to represent the texts as numerical vectors.

## Data Splitting:

I use an 80/20 train/test split to ensure enough data for model training while allowing for robust evaluation.

---

## Chapter 3: Model

### Data Pipeline

#### The Purpose of a Pipeline

- **Streamlining**: Pipelines chain together multiple steps involved in a machine learning process, from data preprocessing to model training. This creates a more organized and reproducible workflow.
- **Modularity**: Easily swap out individual components (e.g., try a different vectorizer or classifier) within the pipeline.
- **Hyperparameter Optimization**: Pipelines integrate seamlessly with tools like GridSearchCV, allowing you to tune parameters across all the steps simultaneously.

#### Components of Pipeline

```python
from sklearn.pipeline import Pipeline
pipeline = Pipeline([
    ('tfidf', TfidfVectorizer(analyzer='word', max_features=1000)),
    ('clf', OneVsRestClassifier(LinearSVC(), n_jobs=1)),
])
```

1. **Feature Engineering**: ('tfidf', TfidfVectorizer(analyzer='word', max_features=1000))

- **TF-IDF**: Stands for Term Frequency - Inverse Document Frequency. This is a classic technique for representing text as numerical vectors. It emphasizes words that are important within a document and rare across the entire collection, helping to identify distinctive terms for each subject area.

- **analyzer='word'**: Tokenizes the text into individual words. This makes sense for general language, but for specialized research papers, you might experiment with character-level or n-gram tokenization.

- **max_features=1000**: Limits the vocabulary to the 1000 most frequent words. This helps control the vector dimensions and can prevent overfitting on rare terms.

2. **Multi-label Classification**: ('clf', OneVsRestClassifier(LinearSVC(), n_jobs=1))

- OneVsRestClassifier: A meta-classifier that adapts traditional binary classifiers (like LinearSVC) to handle multi-label problems. It trains one classifier per possible label in your dataset.

- LinearSVC: A support vector machine for classification. Linear SVCs often work well with text data due to its high-dimensional nature. They are also relatively fast to train and can be interpretable.

- n_jobs=1: Specifies that training should happen on a single CPU core. Increase this if you have more cores available for faster processing.

### Model Choice and Rationale:

Employed LinearSVC within a OneVsRestClassifier for multi-label classification. Linear models are often a good baseline due to their speed and interpretability, making them suitable for understanding text features important for classification.

![model](/img/model.png)

### Hyperparameter Tuning:

- Method: GridSearchCV with Cross-Validation
- Parameter Grid

```python
parameters = {
    'tfidf__max_df': (0.75, 0.85, 0.95),
    'tfidf__min_df': (0.01, 0.05, 0.1),
    'clf__estimator__C': (0.01, 0.1, 1, 10, 100, 1000, 10000)
}
```

Explanation of Parameters

- **tfidf\_\_max_df**: Controls the maximum proportion of documents a term can appear in.
- **tfidf\_\_min_df**: Controls the minimum number of documents a term must appear in.
- **clf**\_\_**estimator**\_\_**C**: Regularization strength in the LinearSVC classifier. Larger values mean less regularization.

- Best Parameters

```python
[('tfidf', TfidfVectorizer(max_df=0.85, max_features=1000, min_df=0.01)), ('clf', OneVsRestClassifier(estimator=LinearSVC(C=100), n_jobs=1))]

```

---

## Chapter 4: Results

### Performance Metrics:

Used F1-score (weighted) and recall (weighted) as they are well-suited for multi-label problems where class imbalance might exist.

**Scores**

F1 score: 0.54

Recall score: 0.50

### Kaggle Screenshot:

![kaggle_result](/img/kaggle.png)

### Interpretation

- Baseline Acknowledgement: The results indicate a functional model that's learned to classify research papers to some degree. However, there's significant room for improvement.

---

## Chapter 5: Discussion

### Error Analysis:

[Link to Folder](https://github.com/tumrabert/kaggle_text_multilable_classification/img/confusion_matrices/)

### Improvement Strategies:

- Alternative Models: Tree-based ensembles (e.g., Random Forest) or Transformers (e.g., BERT) might handle complexities and interactions within the text data better.
- Feature Engineering: Exploring word embeddings or topic modeling could provide richer representations.
- Data Augmentation: If permitted by the competition, techniques to increase dataset size and diversity could boost performance.

## Chapter 6: Conclusion

### Key Findings:

I demonstrated the feasibility of multi-label classification for research papers. Linear models provide a good starting point, but there's potential for significant improvement.

### Challenges:

Obtaining more labeled data was a key challenge.

### Future Directions:

Investigating advanced deep learning models and sophisticated feature engineering techniques would be the next step.
