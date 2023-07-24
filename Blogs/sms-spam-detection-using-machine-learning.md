# SMS Spam Detection Using Machine Learning 

---

---

SMS spam are very common problem. To detect such messages we'll make use of the classification algorithms available in the Scikit Learn and the Natural Language Toolkit python packages. The dataset we will be working on is available at Kaggle [here](https://www.kaggle.com/uciml/sms-spam-collection-dataset).

Before we do the training we'll need to understand the dataset. Let us load up the dataset and look at the contents.
```python
import pandas as pd

sms = pd.read_csv("spam.csv", encoding="latin-1")
sms.head()
```

The *head *method call outputs the first five lines in the DataFrame.

```raw
    v1   v2                                                Unnamed: 2 Unnamed: 3 Unnamed: 4
0   ham  Go until jurong point, crazy.. Available only ...        NaN        NaN        NaN
1   ham                      Ok lar... Joking wif u oni...        NaN        NaN        NaN
2  spam  Free entry in 2 a wkly comp to win FA Cup fina...        NaN        NaN        NaN
3   ham  U dun say so early hor... U c already then say...        NaN        NaN        NaN
4   ham  Nah I don't think he goes to usf, he lives aro...        NaN        NaN        NaN
```

We notice from the output *v1* is the *label *(*spam* or *ham*), *v2 *is the *message *and,* Unnamed: 2*,* Unnamed: 3 *and *Unnamed: 4* doesn't contain any data. As such we will rename *v1 *and *v2* to *label *and *message* and drop the remaining *Unnamed *columns. It is recommended to mobile readers to view the page in landscape mode or to load up the site as desktop site to view the output clearly.

```python
sms = sms.rename(columns={'v1':'label', 'v2':'message'})
sms = sms.drop(['Unnamed: 2', 'Unnamed: 3', 'Unnamed: 4'], axis=1)
```

Now we cannot use the text messages directly in the machine learning algorithm. We need to weight these messages in terms of numbers. Here, natural language processing comes into play. We will be using *Term frequency - Inverse document frequency Vectorizer* (*TfidfVectorizer*) which will extract the significance of each message in terms of floating point values. If you want to understand how the TfidfVectorizer works check out this Quora post [here](https://www.quora.com/How-does-TfidfVectorizer-work-in-laymans-terms#).

But before we get into vectorizer, we will need to perform few text pre-processing. Here's what we'll do:

- Remove punctuations.
- Remove stopwords.
- Remove stemmers.

To remove the punctuations first we create a map of all punctuations pointing to *null *(*None* in python) and then translate the text using that map. To remove all stopwords we will use the *stopwords* library from the *nltk.corpus* package. Stopwords basically are commonly used words (like: *the*, *it*, *and*, etc.) which do not help us tell what the message is about. Let's make a method that perform removal of both and name is as *textPreProcess*.

```python
import string
from nltk.corpus import stopwords

def textPreProcess(text):
    punctuationsToNone = str.maketrans('', '', string.punctuation)
    text = text.translate(punctuationsToNone)
    text = [word for word in text.split() if word.lower() not in stopwords.words("english")]
    return " ".join(text)
```

To remove stemmers we'll be using SnowballStemmer from the *nltk.stem* package. What a stemmer is can be best explained by the following quotation from the tutorial site [here](https://pythonprogramming.net/stemming-nltk-tutorial/).

>"The idea of stemming is a sort of normalizing method. Many variations of words carry the same meaning, other than when tense is involved."

So, basically what we will do is take words like '*include*', '*included*', '*including*' and converted them to a single word like '*include*'. To do this let us create a method named *stemming.*

```python
from nltk.stem import SnowballStemmer

def stemming(text):
    words = map(lambda t: SnowballStemmer("english").stem(t), text.split())
    return " ".join(words)
```

Now that both our functions are ready we can start our text pre-processing. We'll use the *apply* method given by the DataFrame data structure to apply  the methods, *textPreProcess* and *stemming*, to all the messages in the *sms* dataset.

```python
# We make a copy of the message into a new DataFrame
texts = sms['message'].copy()

# And apply the pre-processing methods to the new DataFrame
texts = texts.apply(textPreProcess)
texts = texts.apply(stemming)</pre>
Now, we will use *TfidfVectorizer* to transform the text to feature vectors. This way the data will be useful to the machine learning algorithm we will use.
<pre class="EnlighterJSRAW" data-enlighter-language="python">from sklearn.feature_extraction.text import TfidfVectorizer

vectorizer = TfidfVectorizer("english")
features = vectorizer.fit_transform(texts)
```

Here, we initially not only transform the text to get its features but also fit the *vectorizer* object. This allows us to get the same number of features in the future texts. This is important because our model will learn to classify the mesages as *spam* or *ham* using a set number of features.

Another feature we have not checked yet is the length of the messages. Does the length of the messages help classify a message as *spam* or *ham*? To check this we will plot two histogram graphs to visually check the dependencies.

```python
import matplotlib as mpl
import matplotlib.pyplot as plt
import seaborn as sns

# We create a new column in the DataFrame to store the lengths
# of each message
sms['length'] = sms['message'].apply(len)

mpl.rcParams['patch.force_edgecolor'] = True
plt.style.use('seaborn-bright')
sms.hist(column='length', by='label', bins=50, figsize=(11,5))
```

The output is of the following image:

![image](https://drive.google.com/uc?export=view&id=1Q_pkxAFiECRB_RwJ1ctwQbw0ZoAgKe1u)
Image Ref: https://goo.gl/eEc9DA

From the above images it can inferred that *spam* messages are usually longer than the *ham* messages. Thus the message length can also be a useful feature, so we will be adding the message length to our *features *variable (2D matrix of float64 numbers).

```python
import numpy as np

lengths = sms['length'].values
features = np.hstack((features.todense(), lengths[:, None]))
```

Now that we have the complete set of features ready we can start training our model. There are several classifiers available under the Scikit Learn package, but the one we are going to use here is *Multinomial Naive Bayes* (*MultinomialNB*).

>"The Naive Bayes classifier is a simple probabilistic classifier which is based on Bayes theorem with strong and naïve independence assumptions. It is one of the most basic text classification techniques..." - Source: [DatumBox](http://blog.datumbox.com/machine-learning-tutorial-the-naive-bayes-text-classifier/).

So, since it is based on the Bayes theorem assumption that every pair of feature is independent of each other, it'll be the perfect classifier for our dataset. Let us begin by splitting the dataset into train-test pair and then train the model, test it and finally, calculate the accuracy.

```python
import time
from sklearn.model_selection import train_test_split
from sklearn.naive_bayes import MultinomialNB
from sklearn.metrics import accuracy_score

# Train-Test split: 80-20 percent
features_train, features_test, labels_train, labels_test = train_test_split(
                features, 
                sms['label'], 
                test_size = 0.2, 
                random_state = int(time.time())
)

# Train model
model = MultinomialNB(alpha=0.2)
model.fit(features_train, labels_train)

# Predict and test accuracy
predicted = model.predict(features_test)
accuracyScore = accuracy_score(labels_test, predicted)
print(accuracyScore)
```

The output of the print function is the accuracy score in floating point value which tells us that the classifier predicted with 98.12% accuracy.

```generic
0.9811659192825112
```

In *Multinomial Naive Bayes*, the *alpha* parameter is a hyperparameter, i.e. it controls the form of the model itself. In most cases the best way to find the ideal value of *alpha *is to perform a grid search over possible values and, cross validating the evaluation of performance by the model at each value.

After the above execution the model is ready and now can be used to detect *spams*. One way to go about now is to store the *model* and the *vectorizer* objects to a file so that they can be loaded up in other python applications/scipts to detect spams by bypassing the hassle of training the model again. To do this we can use the *pickle* library.

```python
import pickle

# writing the objects to files as binary
pickle.dump(vectorizer, open("vectorizer.sav", "wb"))
pickle.dump(model, open("model.sav", "wb"))
```

That's it! Now you can load up the *model* and the *vectorizer* objects using *pickle *in any other python application/script and begin detecting spams. Here is a small script to detect if a message is a *spam *or a *ham*.

```python
#!/usr/bin/python

import numpy as np
import pandas as pd
import pickle, string
from nltk.corpus import stopwords
from nltk.stem import SnowballStemmer

def createDataFrame(message):
    return pd.DataFrame({
            'message': [message],
            'length': [len(message)]
    })
    
def textPreProcess(text):
    text = text.translate(str.maketrans('','',string.punctuation))
    text = [word for word in text.split() if word.lower() not in stopwords.words("english")]
    return " ".join(text)

def stemming(text):
    words = map(lambda t: SnowballStemmer('english').stem(t), text.split())
    return " ".join(words)
    
def extractFreatures(sms):
    texts = sms['message'].copy()
    lengths = sms['length'].values
    texts = texts.apply(textPreProcess)
    texts = texts.apply(stemming)
    vectorizer = pickle.load(open('vectorizer.sav', 'rb'))
    features = vectorizer.transform(texts)
    features = np.hstack((features.todense(), lengths[:, None]))
    return features
    
def predict(features):
    model = pickle.load(open('model.sav', 'rb'))
    label = model.predict(features)
    return label

if __name__ == "__main__":
    sms = str(input("Enter the message: "))
    sms = createDataFrame(sms)
    features = extractFreatures(sms)
    label = predict(features)
    print("The message is a " + str(label[0]))
```

The input-output of the above script:

```generic
>> Enter the message: Lucky Draw! Win Exciting Gifts! T&C Applied
The message is a spam
```

This is a small demonstration of one such application that uses the machine learning model. Another application is a Web API made using Flask and RethinkDB available at my Github [here](http://github.com/SpandanBG/SMSSpamWebAPI). Do let me know any suggestion and/or post your queries in the comment section below.

References:
- Dataset: [https://www.kaggle.com/muzzzdy/sms-spam-detection-with-various-classifiers/data](https://www.kaggle.com/uciml/sms-spam-collection-dataset)
- ML Notebook: [https://www.kaggle.com/muzzzdy/sms-spam-detection-with-various-classifiers](https://www.kaggle.com/muzzzdy/sms-spam-detection-with-various-classifiers)
- Stemming: [https://pythonprogramming.net/stemming-nltk-tutorial/](https://pythonprogramming.net/stemming-nltk-tutorial/)
- Naive Bayes: [http://blog.datumbox.com/machine-learning-tutorial-the-naive-bayes-text-classifier/](http://blog.datumbox.com/machine-learning-tutorial-the-naive-bayes-text-classifier/)

---

Spandan Buragohain,
2018-06-22 15:32:09
    