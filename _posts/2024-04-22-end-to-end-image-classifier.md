---
title: "End-to-End Image Classification App"
layout: post
---

The **End-to-End Cancer Classification App** utilizes TensorFlow to train a binary classification model on a dataset of chest CT images. Images are categorized in two classes: adenocarcinoma and normal cases. Using a pre-trained, base model, VGG16, I built a convolutional neural network (CNN) model that assigns images to either class. The model is accessible through a frontend using the Gradio framework, which features an intuitive, user-friendly interface allowing for straightforward interactions, such as image uploads or direct pasting from the web or local storage. Detailed information about this project is available on my [GitHub repository](https://github.com/drjodyannjones/End-to-End-Cancer-Classification-Project){:target="\_blank"}.

### Project Overview

This project was conceived to harness the power of deep learning algorithms for the precise classification of medical imaging, thereby enhancing diagnostic accuracy. One significant application of this model is to support healthcare professionals, such as radiologists, by offering faster and more precise assessments critical during the treatment planning phase.

Beyond its initial application in aiding radiologists, the potential of the project extends to several other fields. For instance, it could be adapted for use in pathology to enhance the detection and analysis of histopathological slides, significantly speeding up the process of diagnosing diseases at the cellular level. Additionally, the model could be utilized in emergency medicine to quickly analyze images in critical care settings, facilitating faster decision-making where time is of the essence. Furthermore, its adaptability means it could also support research in epidemiology by providing insights into disease patterns and prevalence through large-scale image analysis, thereby contributing to public health strategies and preventive medicine. This breadth of applications highlights the versatility and impact of advanced machine learning models in various aspects of healthcare and research.

You may the view the app's screenshot by clicking the following link:
[Screenshot: Chest CT Scan Classifier](https://i.imgur.com/bnbgOpb.png)

### Technologies Employed

- **Python:** I used Python for developing machine learning algorithms and for data manipulation.
- **TensorFlow and Keras:** These tools were essential in building and training the deep learning models.
- **OpenCV:** This library was utilized for processing images, preparing them for the training process.
- **Jupyter Notebook:** I documented the development and testing phases using Jupyter Notebook, which facilitated a clear presentation of the methodologies and results.
- **Gradio:** I used Gradio to create the frontend framework.
- **MLFlow** I used MLFlow for experiment tracking.
- **DVC** I used DVC for data version tracking and control.

## Getting Started

### Installation

Interested parties can replicate or review the project by following these steps:

Step 1. Clone the repository by typing the following in your terminal:

```bash
git clone https://github.com/drjodyannjones/End-to-End-Cancer-Classification-Project.git
```

Step 2. Create a virtual environment.

Step 3. Activate the newly created virtual environment.

Step 4. Navigate to the project directory and install necessary dependencies by typing the following in your terminal:

```bash
pip install -r requirements.txt
```

Step 5. Run the app by typing the following in your terminal

```bash
gradio app.py
```

### Sample Code Snippet: prediction.py

Below is a sample code snippet from the project, illustrating the loading of the trained tensorflow model and the logic used to generate predictions.

{% highlight python %}
import numpy as np
from tensorflow.keras.models import load_model
from tensorflow.keras.preprocessing import image
import os

class PredictionPipeline:
def **init**(self):
self.model = load_model(os.path.join("artifacts", "training", "model.h5"))

    def predict(self, filename):
        try:
            test_image = image.load_img(filename, target_size=(224, 224))
            test_image = image.img_to_array(test_image)
            test_image = np.expand_dims(test_image, axis=0)
            result = np.argmax(self.model.predict(test_image), axis=1)

            if result[0] == 1:
                prediction = "Normal"
            else:
                prediction = "Adenocarcinoma Cancer"

            return [{"image": prediction}]
        except Exception as e:
            print(f"Error during prediction: {e}")
            return [{"image": "Prediction error"}]

{% endhighlight %}

**Acknowledgements**

- Special shoutout to <a href="https://www.youtube.com/watch?v=-NOIWzjJK-4&t=17851s&ab_channel=DSwithBappy">DSwithBappy</a> who inspired this project.
