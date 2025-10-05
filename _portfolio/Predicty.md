---
header:
  overlay_image: assets/images/portfolio/Predicty_dashboard.png.webp
  caption: "Photo credit: [**ChatGPT**]"
permalink: /portfolio/Predicty/
date: 2024-10-08
toc: true
toc_label: "Contents"
excerpt: "Machine learning medical prediction app with 80% accuracy for diabetes risk assessment using Django and Python."
seo_title: "Predicty Medical Prediction Application | Colonneil"
seo_description: "Django-based medical prediction application using machine learning for diagnostic predictions with 80% accuracy in diabetes risk assessment."
---

# Predicty: Medical Prediction Application

**Predicty** is a machine learning-driven web application designed for predictive diagnostics in medical conditions. This project was developed as an educational project for the graduate year of my Bachelor's degree. Built using Python and Django, it showcases the integration of machine learning models into a web-based framework for medical applications, focusing on usability, accuracy, and efficient diagnostics.

[GitHub Repository](https://github.com/abbesachraf/Predicty)

![Predicty Dashboard](/assets/images/portfolio/Predicty_dashboard.png.webp)

### Project Overview

**Objective**: The main goal of Predicty is to assist medical professionals by providing an easy-to-use tool for predicting medical conditions based on patient data. The application supports various machine learning models that analyze inputs to generate predictions with high accuracy.

### Features

- **Prediction Models**: Implements machine learning models to predict the probability of medical conditions, leveraging both classification and regression techniques.
- **Accuracy**: Achieved an **80% prediction accuracy** for diabetes risk assessment, with further enhancements possible through model refinement.
- **Django Framework**: The web app was developed using Django, ensuring robust backend management and seamless integration with Python-based machine learning libraries such as TensorFlow and Scikit-learn.
- **User Interface**: The dashboard provides a user-friendly experience for medical personnel, featuring input forms for patient data and a results panel displaying prediction outcomes.
- **Data Visualization**: Graphs and charts are displayed to show the performance of models and the probability of certain medical outcomes, ensuring clear communication of results to users.

### Key Technologies

- **Backend**: Python, Django
- **Frontend**: HTML, CSS, JavaScript
- **Machine Learning**: Scikit-learn, TensorFlow
- **Database**: SQLite (local) / PostgreSQL (production)
- **Deployment**: Docker for containerization and easy scalability

### Usage

Users can input patient data, such as age, blood pressure, glucose levels, etc., into the web interface. The machine learning model processes the data and returns a probability score indicating the likelihood of the patient developing a medical condition (e.g., diabetes).

### Model Performance

- **Training Dataset**: The model was trained on a dataset of patient records.
- **Accuracy**: **80% prediction accuracy** for diabetes, with ongoing improvements for other conditions.
- **Test Results**: 75% F1 score on test data, with a recall of 78%, indicating a good balance between precision and recall.

### Future Enhancements

- **Model Expansion**: Adding more predictive models to cover additional medical conditions.
- **Accuracy Improvement**: Further tuning of machine learning models and integration of deep learning techniques to increase the accuracy to **85-90%**.
- **API Integration**: Plan to integrate with hospital systems for real-time data input and result retrieval.

### Screenshots

![Predicty Interface](/assets/images/portfolio/Predicty_dashboard.png.webp)

The image above displays the interface of **Predicty**, featuring a patient data input form, a clean dashboard, and prediction results visualized through graphs for better clarity.