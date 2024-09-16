### Project Name:
**Heart Disease Prediction Using Machine Learning**

### Kaggle Dataset Link:
[Heart Disease UCI Dataset](https://www.kaggle.com/datasets/fedesoriano/heart-failure-prediction)

---

### Common Problems in React and Node.js

Here are some challenges you may encounter when building a project to predict heart disease using React for the frontend and Node.js for the backend:

### 1. **Data Preprocessing and Feature Engineering**:
   - **Problem**: The dataset may contain missing values, inconsistent data, or features that need transformation (e.g., encoding categorical features like `Sex` or `ChestPainType`).
     - **React Challenge**: Performing intensive data preprocessing on the frontend can result in poor performance and complexity in handling state.
     - **Node.js Solution**: Handle data preprocessing and feature engineering on the backend. Use Node.js to clean the data and encode categorical variables (e.g., one-hot encoding for `Sex` and `ChestPainType`) before sending it to the frontend for display or model training.
     - **React Solution**: The React frontend should focus on receiving already preprocessed and encoded data from the backend and formatting it for display or form input. You can also handle minor validation on form inputs.

### 2. **Handling Categorical Features**:
   - **Problem**: Categorical features such as `Sex`, `ChestPainType`, `RestingECG`, and `ExerciseAngina` need to be encoded into numerical values for training the classification model.
     - **React Challenge**: Storing and managing state for categorical features and converting them to numerical values for submission to the backend can be tricky.
     - **Node.js Solution**: In Node.js, use feature encoding techniques like label encoding or one-hot encoding for categorical features before passing the data to the machine learning model.
     - **React Solution**: Use dropdown menus or radio buttons in the React frontend for categorical fields (like `Sex` and `ChestPainType`), and send the selected values to the backend as part of the API request.

### 3. **Training a Classification Model**:
   - **Problem**: React cannot handle model training directly since it is a UI framework.
     - **React Challenge**: Implementing machine learning algorithms on the frontend is not feasible due to performance and computational limitations.
     - **Node.js Solution**: Handle model training and inference on the backend using machine learning libraries like `tensorflow.js` or delegate this task to a Python service (using Flask or FastAPI) with `scikit-learn`. Train a classification model (e.g., logistic regression, decision trees, or random forest) on the processed data.
     - **React Solution**: Provide a simple interface in React for users to input patient data (e.g., `Age`, `Sex`, `Cholesterol`), and submit the form to the backend to receive a prediction of heart disease presence.

### 4. **Evaluating Model Performance**:
   - **Problem**: You need to evaluate the classification model’s performance using metrics such as accuracy, precision, recall, and F1-score.
     - **React Challenge**: Displaying model evaluation metrics in a clear, easy-to-understand format can be challenging.
     - **Node.js Solution**: Calculate evaluation metrics on the backend after training the model. Send the evaluation results (e.g., accuracy, confusion matrix) to the frontend for display.
     - **React Solution**: Use charts or summary cards in React to show metrics like accuracy, precision, and recall. For example, use a confusion matrix visualization or bar charts to present the results.

### 5. **Prediction API Design**:
   - **Problem**: You need a well-designed API to send patient health data to the backend and receive predictions (i.e., whether heart disease is present or not).
     - **React Challenge**: Implementing forms and managing state for submitting patient health data to the backend.
     - **Node.js Solution**: Create RESTful API endpoints that accept patient health data (such as `Age`, `Sex`, `Cholesterol`, etc.) from the frontend and pass this data to the classification model for prediction.
     - **React Solution**: Use form components (e.g., `input`, `select`, `radio`) for users to input patient data. Use `axios` or `fetch` to send the data to the backend and receive the prediction. Display the prediction result (e.g., `Heart Disease: Yes/No`) clearly on the frontend.

### 6. **Handling Real-Time User Inputs**:
   - **Problem**: Users might want to adjust patient data (e.g., age, cholesterol level) and see real-time updates or predictions based on the input.
     - **React Challenge**: Handling real-time updates for form inputs and sending requests to the backend could be performance-intensive if not managed well.
     - **Node.js Solution**: Use caching techniques to handle frequent requests or allow for batch processing of requests on the backend for real-time or near-real-time predictions.
     - **React Solution**: Use controlled components for form input fields and send data to the backend after all fields are entered (instead of sending data after every keystroke). Display loading indicators while the backend processes the data.

---





Below is a complete solution for predicting heart disease using the Heart Disease UCI dataset, where we address the problems discussed using React for the frontend and Node.js for the backend. The sample data will be loaded from a local CSV file. I’ll also include the `curl` commands for testing the API and provide the full solution with React components.

### Backend: Node.js with Express

The Node.js backend will:
1. Load the heart disease data from a local CSV file.
2. Clean and preprocess the data.
3. Provide API endpoints for feature encoding, model training, and prediction.

#### 1. **Set Up Node.js Environment**:
Install the required dependencies:
```bash
npm init -y
npm install express csv-parser tensorflow-js
```

#### 2. **Create a `server.js` file** to load the data and provide API endpoints:

```javascript
const express = require('express');
const fs = require('fs');
const csv = require('csv-parser');
const tf = require('@tensorflow/tfjs-node');
const app = express();
app.use(express.json());

let heartDiseaseData = [];

// 1. Load and Clean Data (Problem 1)
fs.createReadStream('./heart_disease.csv')
  .pipe(csv())
  .on('data', (row) => {
    heartDiseaseData.push({
      Age: parseInt(row['Age'], 10),
      Sex: row['Sex'] === 'M' ? 1 : 0,
      ChestPainType: encodeChestPain(row['ChestPainType']),
      RestingBP: parseInt(row['RestingBP'], 10),
      Cholesterol: parseInt(row['Cholesterol'], 10),
      FastingBS: parseInt(row['FastingBS'], 10),
      RestingECG: encodeRestingECG(row['RestingECG']),
      MaxHR: parseInt(row['MaxHR'], 10),
      ExerciseAngina: row['ExerciseAngina'] === 'Y' ? 1 : 0,
      Oldpeak: parseFloat(row['Oldpeak']),
      HeartDisease: parseInt(row['HeartDisease'], 10)
    });
  })
  .on('end', () => {
    console.log('CSV file successfully processed');
  });

// Helper functions for encoding categorical data (Problem 2)
function encodeChestPain(type) {
  switch (type) {
    case 'ATA': return 0;
    case 'NAP': return 1;
    case 'ASY': return 2;
    case 'TA': return 3;
    default: return 0;
  }
}

function encodeRestingECG(ecg) {
  switch (ecg) {
    case 'Normal': return 0;
    case 'ST': return 1;
    case 'LVH': return 2;
    default: return 0;
  }
}

// 2. Feature Engineering (Problem 2)
app.get('/api/feature-selection', (req, res) => {
  const features = heartDiseaseData.map((data) => ({
    Age: data.Age,
    Sex: data.Sex,
    ChestPainType: data.ChestPainType,
    RestingBP: data.RestingBP,
    Cholesterol: data.Cholesterol,
    FastingBS: data.FastingBS,
    RestingECG: data.RestingECG,
    MaxHR: data.MaxHR,
    ExerciseAngina: data.ExerciseAngina,
    Oldpeak: data.Oldpeak,
  }));

  res.json(features);
});

// 3. Model Training (Problem 3)
app.post('/api/train-model', (req, res) => {
  const { features, labels } = req.body;

  const featureTensor = tf.tensor2d(features);
  const labelTensor = tf.tensor2d(labels);

  const model = tf.sequential();
  model.add(tf.layers.dense({ units: 1, inputShape: [10], activation: 'sigmoid' }));

  model.compile({ optimizer: 'adam', loss: 'binaryCrossentropy', metrics: ['accuracy'] });

  model.fit(featureTensor, labelTensor, { epochs: 50 }).then((history) => {
    res.json({ message: 'Model trained successfully', accuracy: history.history.acc });
  });
});

// 4. Predict Heart Disease (Problem 5)
app.post('/api/predict-heart-disease', (req, res) => {
  const { features } = req.body;
  const featureTensor = tf.tensor2d([features]);

  const model = tf.sequential();
  model.add(tf.layers.dense({ units: 1, inputShape: [10], activation: 'sigmoid' }));

  const prediction = model.predict(featureTensor);
  prediction.data().then((predicted) => {
    res.json({ predictedHeartDisease: predicted[0] > 0.5 ? 'Yes' : 'No' });
  });
});

// Start the server
app.listen(5000, () => {
  console.log('Server is running on http://localhost:5000');
});
```

#### Sample Data in CSV (heart_disease.csv):
```csv
Age,Sex,ChestPainType,RestingBP,Cholesterol,FastingBS,RestingECG,MaxHR,ExerciseAngina,Oldpeak,HeartDisease
40,M,ATA,140,289,0,Normal,172,N,0,0
```

---

### Frontend: React Application

#### 1. **Set up the React app**:
Create a new React app if you don't have one already:

```bash
npx create-react-app heart-disease-prediction
cd heart-disease-prediction
npm install axios recharts
```

#### 2. **Feature Selection Component**: `FeatureSelection.js`

This component fetches and displays the selected features from the backend.

```jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const FeatureSelection = () => {
  const [features, setFeatures] = useState([]);

  useEffect(() => {
    const fetchFeatures = async () => {
      try {
        const response = await axios.get('http://localhost:5000/api/feature-selection');
        setFeatures(response.data);
      } catch (error) {
        console.error('Error fetching features', error);
      }
    };

    fetchFeatures();
  }, []);

  return (
    <div>
      <h3>Selected Features for Heart Disease Prediction</h3>
      <ul>
        {features.map((feature, index) => (
          <li key={index}>
            Age: {feature.Age}, Sex: {feature.Sex}, Chest Pain Type: {feature.ChestPainType}, 
            Resting BP: {feature.RestingBP}, Cholesterol: {feature.Cholesterol}
          </li>
        ))}
      </ul>
    </div>
  );
};

export default FeatureSelection;
```

#### 3. **Model Training Component**: `ModelTraining.js`

This component allows the user to send features and labels for model training.

```jsx
import React, { useState } from 'react';
import axios from 'axios';

const ModelTraining = () => {
  const [message, setMessage] = useState('');

  const handleTrainModel = async () => {
    const features = [
      [40, 1, 0, 140, 289, 0, 0, 172, 0, 0], // Example feature data
    ];
    const labels = [
      [0], // Example label data (heart disease presence)
    ];

    try {
      const response = await axios.post('http://localhost:5000/api/train-model', { features, labels });
      setMessage(response.data.message);
    } catch (error) {
      console.error('Error training model', error);
    }
  };

  return (
    <div>
      <h3>Train Heart Disease Prediction Model</h3>
      <button onClick={handleTrainModel}>Train Model</button>
      <p>{message}</p>
    </div>
  );
};

export default ModelTraining;
```

#### 4. **Heart Disease Prediction Component**: `HeartDiseasePrediction.js`

This component allows users to input health data and get a prediction for heart disease presence.

```jsx
import React, { useState } from 'react';
import axios from 'axios';

const HeartDiseasePrediction = () => {
  const [predictedHeartDisease, setPredictedHeartDisease] = useState(null);
  const [features, setFeatures] = useState([40, 1, 0, 140, 289, 0, 0, 172, 0, 0]); // Example feature input

  const handlePredictHeartDisease = async () => {
    try {
      const response = await axios.post('http://localhost:5000/api/predict-heart-disease', { features });
      setPredictedHeartDisease(response.data.predictedHeartDisease);
    } catch (error) {
      console.error('Error predicting heart disease', error);
    }
  };

  return (
    <div>
      <h3>Predict Heart Disease</h3>
      <button onClick={handlePredictHeartDisease}>Predict</button>
      {predictedHeartDisease && <p>Heart Disease: {predictedHeartDisease}</p>}
    </div>
  );
};

export default HeartDiseasePrediction;
```

---

### `curl` Commands to Test the Backend:

1. **Feature Selection** (Problem 2)
```bash
curl -X GET http://localhost:5000/api/feature-selection
```

Here is the continuation of the solution, including the `curl` commands for testing and the rest of the application components.

---

### `curl` Commands to Test the Backend (Continued):

2. **Model Training** (Problem 3) (continued):
```bash
curl -X POST http://localhost:5000/api/train-model \
-H "Content-Type: application/json" \
-d '{
  "features": [[40, 1, 0, 140, 289, 0, 0, 172, 0, 0]],
  "labels": [[0]]
}'
```

3. **Heart Disease Prediction** (Problem 5):
```bash
curl -X POST http://localhost:5000/api/predict-heart-disease \
-H "Content-Type: application/json" \
-d '{"features": [40, 1, 0, 140, 289, 0, 0, 172, 0, 0]}'
```

---

### Frontend: React Application (Continued)

#### 5. **Complete Application**

Now, we’ll combine the components into the main `App.js` file to provide a full UI for feature selection, model training, and heart disease prediction.

```jsx
import React from 'react';
import FeatureSelection from './components/FeatureSelection';
import ModelTraining from './components/ModelTraining';
import HeartDiseasePrediction from './components/HeartDiseasePrediction';

const App = () => {
  return (
    <div>
      <h1>Heart Disease Prediction Dashboard</h1>
      <FeatureSelection />
      <ModelTraining />
      <HeartDiseasePrediction />
    </div>
  );
};

export default App;
```

---

### Running the Project

#### 1. Backend (Node.js):
- Save your `heart_disease.csv` file in the root folder.
- Start the Node.js server:
```bash
node server.js
```

#### 2. Frontend (React):
- Run the React app:
```bash
npm start
```

---

### Explanation of Solution

1. **Backend (Node.js)**:
   - **Problem 1 & 2**: The backend loads the heart disease dataset from a CSV file, encodes categorical features (like `ChestPainType`, `Sex`, etc.), and makes them ready for training and prediction.
   - **Problem 3**: Model training is handled on the backend using TensorFlow.js. It takes the processed features and labels and trains a simple binary classification model (e.g., logistic regression) to predict heart disease.
   - **Problem 5**: The backend provides a prediction API where users can send patient health data (features) and receive a prediction of heart disease presence.

2. **Frontend (React)**:
   - **Problem 2**: The `FeatureSelection` component fetches and displays the processed feature data for each patient, which helps users understand what data is being used for training and prediction.
   - **Problem 3**: The `ModelTraining` component allows the user to trigger model training on the backend by sending feature and label data.
   - **Problem 5**: The `HeartDiseasePrediction` component allows users to input patient health data and get a prediction from the backend API. This component displays whether heart disease is predicted or not.

---

### Summary of Components:
- **FeatureSelection.js**: Fetches and displays processed features.
- **ModelTraining.js**: Sends features and labels to the backend for model training.
- **HeartDiseasePrediction.js**: Allows users to input patient data and predict heart disease presence.
- **App.js**: Combines the above components to create a full dashboard for heart disease prediction.





In the line:

```js
const labels = [
  [0], // Example label data (heart disease presence)
];
```

This represents the label (or target value) for the machine learning model. In the context of heart disease prediction:

- `0`: Represents that the patient **does not** have heart disease.
- `1`: Represents that the patient **has** heart disease.

In this case, the label `[0]` means that the patient corresponding to the provided feature set does **not** have heart disease. When training the model, this label will be compared against the model's predictions to compute the loss and update the model accordingly.

You would typically have multiple labels corresponding to the features you provide. For example, if you're training on multiple samples (patients), your `labels` array would look like this:

```js
const labels = [
  [0],  // Patient 1: No heart disease
  [1],  // Patient 2: Has heart disease
  [1],  // Patient 3: Has heart disease
  [0],  // Patient 4: No heart disease
];
```

In short, this project is **predicting whether a person has heart disease or not** based on their health data, such as:

- Age
- Gender
- Chest pain type
- Blood pressure
- Cholesterol level
- Maximum heart rate
- And other health-related features

Using this data, the system will analyze patterns and predict if a person is at risk of heart disease. We’re using machine learning to **train a model** to learn these patterns, and once trained, it can take new patient data and predict whether heart disease is likely present or not.

In simpler terms, **we're teaching the computer to recognize signs of heart disease** using past health data, so it can help doctors make predictions for new patients.

