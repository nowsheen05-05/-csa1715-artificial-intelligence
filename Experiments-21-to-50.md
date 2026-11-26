# CSA17 Artificial Intelligence Lab — Experiments 21–50

This bundle contains the next 30 experiments from the five-day CSA17 lab list (Day 3: 21–30, Day 4: 31–40, Day 5: 41–50). Each section has an aim, runnable example, and sample result. Python dependencies are listed beside the relevant program. Install them with `python -m pip install <package>`. Prolog programs can be run in SWI-Prolog.

> Sample outputs are illustrative; library versions, random seeds, training time and input files can affect results. The diet and medical-rule examples are logic demonstrations, not health advice.

---

## Day 3 — Prolog (Experiments 21–30)

### 21. Towers of Hanoi
**Aim:** Solve the Towers of Hanoi recursively.

```prolog
move(1, From, To, _) :-
    format('Move disk 1 from ~w to ~w~n', [From, To]).
move(N, From, To, Aux) :-
    N > 1, M is N - 1,
    move(M, From, Aux, To),
    format('Move disk ~w from ~w to ~w~n', [N, From, To]),
    move(M, Aux, To, From).
hanoi(N, From, To, Aux) :-
    integer(N), N > 0, move(N, From, To, Aux).
```
**Query:** `?- hanoi(3, left, right, middle).`
**Sample result:** Seven legal moves are printed; the final move transfers disk 1 from middle to right.

### 22. Check whether a particular bird can fly
**Aim:** Use facts and an exception rule to answer flight queries.

```prolog
bird(sparrow). bird(pigeon). bird(eagle).
bird(ostrich). bird(penguin).
cannot_fly(ostrich). cannot_fly(penguin).
can_fly(Bird) :- bird(Bird), \+ cannot_fly(Bird).
```
**Queries:** `?- can_fly(sparrow).` → `true`; `?- can_fly(penguin).` → `false`.
**Result:** The rule infers flight ability except for birds listed as unable to fly.

### 23. Family tree
**Aim:** Represent family facts and infer relationships.

```prolog
parent(john, mary). parent(john, david).
parent(susan, mary). parent(susan, david).
parent(mary, alice). parent(robert, alice).
male(john). male(david). male(robert).
female(susan). female(mary). female(alice).
father(F,C) :- parent(F,C), male(F).
mother(M,C) :- parent(M,C), female(M).
grandparent(G,C) :- parent(G,P), parent(P,C).
sibling(X,Y) :- parent(P,X), parent(P,Y), X \= Y.
```
**Queries:** `?- grandparent(john, alice).` → `true`; `?- sibling(mary, david).` → `true`.
**Result:** Family relationships are inferred from parent and gender facts.

### 24. Diet suggestion based on disease (rule demonstration)
**Aim:** Demonstrate a rule-based suggestion system. This is not medical advice.

```prolog
suggestion(diabetes, 'Discuss a balanced, high-fibre meal plan with a dietitian').
suggestion(hypertension, 'Discuss lower-sodium choices with a clinician').
suggestion(anemia, 'Ask a clinician about iron status and suitable foods').
suggestion(general, 'Choose varied meals with vegetables, fruit, grains and protein').
diet(Condition, Advice) :- suggestion(Condition, Advice).
```
**Query:** `?- diet(diabetes, Advice).`
**Sample result:** Advice = 'Discuss a balanced, high-fibre meal plan with a dietitian'.
**Result:** Prolog returns the text associated with a known condition label.

### 25. Monkey and Banana problem
**Aim:** Find a plan in a simplified state-space model.

```prolog
% state(MonkeyPosition, BoxPosition, HasBanana)
step(state(door, Box, no), walk_to_middle, state(middle, Box, no)).
step(state(middle, Box, no), push_box, state(middle, middle, no)) :-
    Box \= middle.
step(state(middle, middle, no), climb_box, state(on_box, middle, no)).
step(state(on_box, middle, no), grasp_banana, state(on_box, middle, yes)).
solve(S, [], S) :- S = state(_, _, yes).
solve(S, [A|Rest], Final) :-
    step(S, A, Next), solve(Next, Rest, Final).
```
**Query:** `?- solve(state(door, corner, no), Plan, Final).`
**Sample result:** Plan = [walk_to_middle, push_box, climb_box, grasp_banana].
**Result:** The sequence of legal state transitions reaches the goal state.

### 26. Fruit and colour using backtracking
**Aim:** Use Prolog backtracking to find all matching fruit-colour facts.

```prolog
fruit_color(apple, red).
fruit_color(banana, yellow).
fruit_color(grape, purple).
fruit_color(orange, orange).
fruit_color(guava, green).
```
**Queries:** `?- fruit_color(apple, C).` → `C = red`; `?- fruit_color(F, yellow).` → `F = banana`.
**Result:** Query `fruit_color(F,C)` and press `;` to view further solutions.

### 27. Best First Search
**Aim:** Demonstrate heuristic-guided graph search.

```prolog
edge(a,b). edge(a,c). edge(b,d). edge(c,e). edge(d,f). edge(e,f).
heuristic(a,6). heuristic(b,4). heuristic(c,2).
heuristic(d,3). heuristic(e,1). heuristic(f,0).
best_first(Start, Goal, Path) :-
    search([Start], Goal, [], Rev), reverse(Rev, Path).
search([Goal|_], Goal, Visited, [Goal|Visited]).
search([Node|Rest], Goal, Visited, Path) :-
    findall(H-Next,
      (edge(Node,Next), \+ member(Next,[Node|Visited]), heuristic(Next,H)),
      Pairs),
    append(Rest, Pairs, Queue0), keysort(Queue0, Sorted),
    values(Sorted, Queue), search(Queue, Goal, [Node|Visited], Path).
values([], []).
values([_-V|T], [V|R]) :- values(T,R).
```
**Query:** `?- best_first(a, f, Path).`
**Result:** Returns a heuristic-guided path when the goal is reachable. (This compact teaching example uses a small graph and a simple frontier.)

### 28. Medical diagnosis using rules
**Aim:** Demonstrate logical inference from symptom facts; it is not a diagnostic tool.

```prolog
symptom(case1, fever). symptom(case1, cough).
symptom(case2, sneezing). symptom(case2, itchy_eyes).
possible_label(P, 'cold-like symptom pattern') :-
    symptom(P, fever), symptom(P, cough).
possible_label(P, 'allergy-like symptom pattern') :-
    symptom(P, sneezing), symptom(P, itchy_eyes).
```
**Query:** `?- possible_label(case1, Result).`
**Sample result:** Result = 'cold-like symptom pattern'.
**Result:** The rules match stored facts to illustrative labels only.

### 29. Forward chaining
**Aim:** Infer conclusions by applying rules to known facts.

```prolog
rule(wet_ground, [rain]).
rule(traffic_slow, [wet_ground]).
derive(X, Known) :- member(X, Known).
derive(X, Known) :-
    rule(X, Conditions), all_known(Conditions, Known).
all_known([], _).
all_known([H|T], Known) :- derive(H, Known), all_known(T, Known).
```
**Query:** `?- derive(traffic_slow, [rain]).` → `true`.
**Result:** The conclusion is inferred from the initial fact through intermediate rules.

### 30. Backward chaining
**Aim:** Prove a goal by recursively checking its supporting facts and rules.

```prolog
fact(rain).
rule(wet_ground, [rain]).
rule(traffic_slow, [wet_ground]).
prove(X) :- fact(X).
prove(X) :- rule(X, Conditions), prove_all(Conditions).
prove_all([]).
prove_all([H|T]) :- prove(H), prove_all(T).
```
**Query:** `?- prove(traffic_slow).` → `true`.
**Result:** Prolog starts from the goal and works backwards to the known fact.

---

## Day 4 — Web and Machine Learning (Experiments 31–40)

### 31. Create a WordPress blog demonstrating Anchor Tag, Title Tag, etc.
**Aim:** Create a simple WordPress blog and demonstrate HTML title and anchor tags.

**Procedure:** Open the WordPress site supplied by your institution → Dashboard → Posts → Add New. Enter the post title **AI Lab Blog**, add a heading and paragraph, then add a **Custom HTML** block containing:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>AI Lab Blog</title>
</head>
<body>
  <h1>Artificial Intelligence Lab</h1>
  <p>Welcome to my learning blog.</p>
  <a href="https://wordpress.org/" target="_blank" rel="noopener">
    Learn about WordPress
  </a>
</body>
</html>
```
Preview the post and verify the heading and hyperlink. Publish only if permitted by your instructor.
**Sample result:** Browser tab title is “AI Lab Blog”; the page shows the heading and a working WordPress link.
**Result:** A basic blog post demonstrates the requested title and anchor elements.

### 32. Linear regression
**Aim:** Predict a continuous value using a fitted linear model. Install: `pip install scikit-learn`.

```python
import numpy as np
from sklearn.linear_model import LinearRegression

X = np.array([[1], [2], [3], [4], [5]])
y = np.array([2, 4, 6, 8, 10])
model = LinearRegression().fit(X, y)
print("Slope:", model.coef_[0])
print("Intercept:", model.intercept_)
print("Prediction for x=6:", model.predict([[6]])[0])
```
**Sample output:** Slope: 2.0; Intercept: 0.0; Prediction for x=6: 12.0.
**Result:** The trained regression model predicts a continuous target.

### 33. K-nearest neighbors classification
**Aim:** Classify Iris flowers using nearby training samples. Install: `pip install scikit-learn`.

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import accuracy_score

iris = load_iris()
Xtr, Xte, ytr, yte = train_test_split(
    iris.data, iris.target, test_size=0.25, random_state=42, stratify=iris.target)
model = KNeighborsClassifier(n_neighbors=5).fit(Xtr, ytr)
pred = model.predict(Xte)
print("First predicted class:", iris.target_names[pred[0]])
print("Accuracy:", round(accuracy_score(yte, pred), 2))
```
**Sample result:** A flower class name and an accuracy score between 0 and 1 are printed.
**Result:** KNN predicts classes based on the nearest training examples.

### 34. Neural network with TensorFlow for image classification
**Aim:** Train a neural network on Fashion-MNIST. Install: `pip install tensorflow`.

```python
import tensorflow as tf

(x_train, y_train), (x_test, y_test) = tf.keras.datasets.fashion_mnist.load_data()
x_train, x_test = x_train / 255.0, x_test / 255.0
model = tf.keras.Sequential([
    tf.keras.layers.Flatten(input_shape=(28, 28)),
    tf.keras.layers.Dense(128, activation="relu"),
    tf.keras.layers.Dropout(0.2),
    tf.keras.layers.Dense(10, activation="softmax")
])
model.compile(optimizer="adam",
              loss="sparse_categorical_crossentropy", metrics=["accuracy"])
model.fit(x_train, y_train, epochs=3, validation_split=0.1)
loss, accuracy = model.evaluate(x_test, y_test, verbose=0)
print("Test accuracy:", round(float(accuracy), 4))
```
**Sample result:** Prints test accuracy (varies by run and environment).
**Result:** The network learns to classify images into ten clothing categories.

### 35. Decision tree classification
**Aim:** Classify Iris flowers with a decision tree. Install: `pip install scikit-learn`.

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.tree import DecisionTreeClassifier
from sklearn.metrics import accuracy_score

iris = load_iris()
Xtr, Xte, ytr, yte = train_test_split(
    iris.data, iris.target, test_size=0.2, random_state=7, stratify=iris.target)
model = DecisionTreeClassifier(max_depth=3, random_state=7).fit(Xtr, ytr)
print("Prediction:", iris.target_names[model.predict([Xte[0]])[0]])
print("Accuracy:", round(accuracy_score(yte, model.predict(Xte)), 2))
```
**Sample result:** Prints one Iris class and test accuracy.
**Result:** The decision tree learns feature-based classification rules.

### 36. Sentiment analysis using NLP
**Aim:** Classify short text as positive or negative. Install: `pip install scikit-learn`.

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB

texts = ["I love this product", "This is wonderful", "Very happy",
         "I hate this", "This is terrible", "Very disappointed"]
labels = ["positive", "positive", "positive", "negative", "negative", "negative"]
vectorizer = TfidfVectorizer()
X = vectorizer.fit_transform(texts)
model = MultinomialNB().fit(X, labels)
for sentence in ["I love this", "This is terrible"]:
    print(sentence, "->", model.predict(vectorizer.transform([sentence]))[0])
```
**Sample output:** I love this → positive; This is terrible → negative.
**Result:** TF-IDF converts text to features and Naive Bayes predicts sentiment.

### 37. Genetic algorithm for function optimization
**Aim:** Maximize f(x)=x(10−x) for integer x from 0 to 10.

```python
import random
random.seed(7)
def fitness(x):
    return x * (10 - x)
population = [random.randint(0, 10) for _ in range(8)]
for _ in range(30):
    population.sort(key=fitness, reverse=True)
    parents = population[:4]
    children = []
    while len(children) < 4:
        a, b = random.sample(parents, 2)
        child = round((a + b) / 2)
        if random.random() < 0.2:
            child = random.randint(0, 10)
        children.append(child)
    population = parents + children
best = max(population, key=fitness)
print("Best x:", best, "Fitness:", fitness(best))
```
**Expected optimum:** x=5, fitness=25.
**Result:** Selection, crossover and mutation search for a high-fitness solution.

### 38. Support Vector Machine binary classification
**Aim:** Classify two classes using an SVM. Install: `pip install scikit-learn`.

```python
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.svm import SVC
from sklearn.metrics import accuracy_score

data = load_breast_cancer()
Xtr, Xte, ytr, yte = train_test_split(
    data.data, data.target, test_size=0.25, random_state=42, stratify=data.target)
model = make_pipeline(StandardScaler(), SVC(kernel="linear"))
model.fit(Xtr, ytr)
print("Accuracy:", round(accuracy_score(yte, model.predict(Xte)), 3))
```
**Sample result:** Prints classification accuracy. This public teaching dataset is not for diagnosis.
**Result:** SVM learns a separating boundary between two classes.

### 39. Principal Component Analysis (PCA)
**Aim:** Reduce Iris data from four features to two components. Install: `pip install scikit-learn`.

```python
from sklearn.datasets import load_iris
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA

data = load_iris()
scaled = StandardScaler().fit_transform(data.data)
pca = PCA(n_components=2)
reduced = pca.fit_transform(scaled)
print("Original shape:", data.data.shape)
print("Reduced shape:", reduced.shape)
print("Explained variance:", pca.explained_variance_ratio_.round(3))
```
**Sample output:** Original shape: (150, 4); Reduced shape: (150, 2).
**Result:** PCA represents the data using two components while retaining variance.

### 40. Collaborative filtering recommendation system
**Aim:** Recommend items using similarity between users' ratings.

```python
import numpy as np
ratings = np.array([[5,4,0,0], [4,5,0,0],
                    [0,0,5,4], [0,0,4,5]], dtype=float)
user = 0
similarity = np.zeros(len(ratings))
for i, row in enumerate(ratings):
    if i == user:
        continue
    mask = (ratings[user] > 0) & (row > 0)
    if mask.any():
        a, b = ratings[user, mask], row[mask]
        similarity[i] = np.dot(a,b)/(np.linalg.norm(a)*np.linalg.norm(b))
neighbor = int(np.argmax(similarity))
items = np.where((ratings[user] == 0) & (ratings[neighbor] > 0))[0]
print("Nearest user:", neighbor)
print("Recommended item indices:", items.tolist())
```
**Sample result:** The program prints the most similar user and item indices recommended.
**Result:** Items rated by a similar user are suggested to the target user.

---

## Day 5 — AI and Machine Learning (Experiments 41–50)

### 41. Simple chatbot using NLP techniques
**Aim:** Build a rule-based chatbot that responds to common phrases.

```python
def reply(message):
    text = message.lower().strip()
    if any(word in text for word in ["hello", "hi", "hey"]):
        return "Hello! How can I help you?"
    if "your name" in text:
        return "I am a simple Python chatbot."
    if "help" in text:
        return "You can greet me or ask my name."
    if text in {"bye", "exit", "quit"}:
        return "Goodbye!"
    return "Sorry, I did not understand. Please rephrase."

print("Bot:", reply("Hello"))
print("Bot:", reply("What is your name?"))
```
**Sample output:** Bot: Hello! How can I help you? / Bot: I am a simple Python chatbot.
**Result:** The chatbot maps recognized text patterns to responses.

### 42. K-means clustering
**Aim:** Group Iris samples into three clusters. Install: `pip install scikit-learn`.

```python
from sklearn.datasets import load_iris
from sklearn.cluster import KMeans

data = load_iris()
model = KMeans(n_clusters=3, random_state=42, n_init=10)
labels = model.fit_predict(data.data)
print("First 10 cluster labels:", labels[:10].tolist())
print("Cluster-centre shape:", model.cluster_centers_.shape)
```
**Sample result:** Ten cluster labels and centre shape (3, 4) are printed.
**Result:** K-means groups samples by feature similarity.

### 43. Reinforcement learning agent for a basic environment
**Aim:** Train a small Q-learning agent to reach a goal state.

```python
import numpy as np
import random
random.seed(1)
Q = np.zeros((5, 3))  # states 0..4; actions: left, right, stay
alpha, gamma = 0.5, 0.9
for episode in range(500):
    state = 0
    for _ in range(12):
        action = random.randrange(3) if random.random() < 0.2 else int(np.argmax(Q[state]))
        next_state = max(0, min(4, state + [-1, 1, 0][action]))
        reward = 10 if next_state == 4 else -1
        Q[state, action] += alpha * (
            reward + gamma * np.max(Q[next_state]) - Q[state, action])
        state = next_state
        if state == 4:
            break
print("Best action from start:", int(np.argmax(Q[0])))
print("Q-table shape:", Q.shape)
```
**Sample result:** Prints a learned action index and Q-table shape (5, 3).
**Result:** The agent updates action values from rewards and learns to reach the goal.

### 44. Image segmentation using computer vision
**Aim:** Segment bright and dark regions using thresholding. Install: `pip install opencv-python`.

```python
import sys
import cv2
if len(sys.argv) < 2:
    raise SystemExit("Usage: python program.py input.jpg")
image = cv2.imread(sys.argv[1])
if image is None:
    raise SystemExit("Could not read image file")
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
_, mask = cv2.threshold(gray, 127, 255, cv2.THRESH_BINARY)
cv2.imwrite("segmented_mask.png", mask)
print("Saved segmented_mask.png")
```
**Input:** An image file, for example `sample.jpg`. **Output:** `segmented_mask.png`.
**Result:** A binary mask separates pixels above and below the selected threshold.

### 45. Markov Chain model
**Aim:** Predict event-state probabilities using a transition matrix.

```python
import numpy as np
transition = np.array([[0.8, 0.2], [0.4, 0.6]])  # Sunny/Rainy
current = np.array([1.0, 0.0])  # initial state: Sunny
for day in range(1, 4):
    current = current @ transition
    print(f"Day {day}: Sunny={current[0]:.3f}, Rainy={current[1]:.3f}")
```
**Sample output:** Day 1: Sunny=0.800, Rainy=0.200.
**Result:** The state distribution changes according to the transition probabilities.

### 46. Object detection using a pretrained model
**Aim:** Detect object classes in an image using pretrained YOLO weights. Install: `pip install ultralytics`.

```python
import sys
from ultralytics import YOLO
if len(sys.argv) < 2:
    raise SystemExit("Usage: python program.py image.jpg")
model = YOLO("yolov8n.pt")  # pretrained weights; first run may download them
results = model(sys.argv[1])
for result in results:
    for box in result.boxes:
        class_id = int(box.cls[0])
        confidence = float(box.conf[0])
        print(model.names[class_id], round(confidence, 2))
    result.save(filename="detected_output.jpg")
print("Saved detected_output.jpg")
```
**Input:** An image you have permission to process. **Output:** Detected labels, confidence scores and `detected_output.jpg`.
**Result:** The pretrained model identifies supported object categories and draws bounding boxes.

### 47. Hyperparameter tuning
**Aim:** Select SVM parameters using GridSearchCV. Install: `pip install scikit-learn`.

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import GridSearchCV
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.svm import SVC

data = load_iris()
pipe = make_pipeline(StandardScaler(), SVC())
grid = GridSearchCV(pipe,
    {"svc__C": [0.1, 1, 10], "svc__kernel": ["linear", "rbf"]}, cv=5)
grid.fit(data.data, data.target)
print("Best parameters:", grid.best_params_)
print("Best CV score:", round(grid.best_score_, 3))
```
**Sample result:** Prints the best C/kernel combination and cross-validation score.
**Result:** GridSearchCV compares parameter combinations and selects the highest scoring model.

### 48. Bayesian network / probabilistic relationships
**Aim:** Calculate a posterior probability for a simple Rain → WetGrass network.

```python
p_rain = 0.2
p_wet_given_rain = 0.9
p_wet_given_no_rain = 0.1
p_wet = p_wet_given_rain*p_rain + p_wet_given_no_rain*(1-p_rain)
p_rain_given_wet = p_wet_given_rain*p_rain / p_wet
print("P(WetGrass):", round(p_wet, 3))
print("P(Rain | WetGrass):", round(p_rain_given_wet, 3))
```
**Sample output:** P(WetGrass): 0.26; P(Rain | WetGrass): 0.692.
**Result:** Bayes' theorem updates the probability of rain after observing wet grass.

### 49. Face recognition using a pretrained model (consent-based classroom demo)
**Aim:** Compare two face encodings with a pretrained library. Install: `pip install face-recognition`.
**Privacy:** Only use images of people who have explicitly agreed. Do not use this exercise for surveillance, access control or decisions about people. It is not identity verification.

```python
import sys
import face_recognition
if len(sys.argv) != 3:
    raise SystemExit("Usage: python program.py known.jpg test.jpg")
known_img = face_recognition.load_image_file(sys.argv[1])
test_img = face_recognition.load_image_file(sys.argv[2])
known = face_recognition.face_encodings(known_img)
test = face_recognition.face_encodings(test_img)
if not known or not test:
    raise SystemExit("A face could not be encoded in one of the images")
distance = face_recognition.face_distance([known[0]], test[0])[0]
print("Encoding distance:", round(float(distance), 3))
print("Similar under illustrative threshold:", bool(distance < 0.6))
```
**Input:** Two consented sample images. **Output:** Encoding distance and an illustrative similarity flag.
**Result:** The program compares numerical encodings; a threshold is not proof of identity.

### 50. Genetic algorithm to evolve a neural network for XOR
**Aim:** Evolve weights of a small 2–2–1 neural network to approximate XOR.

```python
import numpy as np
import random
random.seed(4)
np.random.seed(4)
X = np.array([[0,0], [0,1], [1,0], [1,1]], dtype=float)
y = np.array([[0], [1], [1], [0]], dtype=float)
def sigmoid(z):
    return 1 / (1 + np.exp(-np.clip(z, -30, 30)))
def predict(g):
    w1, b1 = g[:4].reshape(2,2), g[4:6]
    w2, b2 = g[6:8].reshape(2,1), g[8:9]
    return sigmoid(sigmoid(X @ w1 + b1) @ w2 + b2)
def fitness(g):
    return -float(np.mean((predict(g) - y) ** 2))
population = [np.random.uniform(-2, 2, 9) for _ in range(40)]
for _ in range(300):
    population.sort(key=fitness, reverse=True)
    next_population = [p.copy() for p in population[:6]]
    while len(next_population) < 40:
        a, b = random.sample(population[:20], 2)
        child = np.where(np.random.rand(9) < 0.5, a, b).copy()
        child += np.random.normal(0, 0.2, 9)
        next_population.append(child)
    population = next_population
best = max(population, key=fitness)
print("Best mean squared error:", round(-fitness(best), 4))
print("XOR predictions:", np.round(predict(best).ravel(), 2).tolist())
```
**Sample result:** Prints the best mean-squared error and four XOR predictions (results vary by run).
**Result:** The genetic algorithm evolves neural-network parameters to reduce XOR prediction error.

---

## Notes for running the experiments
- Save each Python code block as a separate `.py` file; save each Prolog block as a `.pl` file.
- Run Python with `python filename.py`; run Prolog in SWI-Prolog with `swipl filename.pl`, then enter the query shown.
- Experiment 31 requires a WordPress site and dashboard access; the HTML is a demonstration to add through a Custom HTML block.
- Some experiments require an internet connection to download public datasets or pretrained weights on first run.
