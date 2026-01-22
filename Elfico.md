~~~
<meta property="og:title" content="Entrenando una red neuronal para reconocer caracteres élficos">
<meta property="og:description" content="Spoiler: Sale mal.">
<meta property="og:image" content="https://upload.wikimedia.org/wikipedia/commons/thumb/e/eb/One_Ring_inscription_Atul.svg/2560px-One_Ring_inscription_Atul.svg.png">
<meta property="og:url" content="https://elfofrit.com/Elfico">
<meta property="og:type" content="article">

<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Entrenando una red neuronal para reconocer caracteres élficos">
<meta name="twitter:description" content="Spoiler: Sale mal.">
<meta name="twitter:image" content="https://upload.wikimedia.org/wikipedia/commons/thumb/e/eb/One_Ring_inscription_Atul.svg/2560px-One_Ring_inscription_Atul.svg.png">
~~~

+++
title = "Entrenando una red neuronal para reconocer caracteres élficos"
hascode = true
date = Date(2025, 4, 7)
rss = "Spoiler: Sale mal."

tags = ["data_science", "stem"]

description = "Caracter en tengwar."
image = "https://upload.wikimedia.org/wikipedia/commons/thumb/e/eb/One_Ring_inscription_Atul.svg/2560px-One_Ring_inscription_Atul.svg.png"
+++

**(Página en construcción)**

# Detección y traducción de caracteres élficos por redes neuronales

Una de las muchas obras que hizo Fëanor durante su estancia en Valinor bajo la luz de los grandes árboles fue el diseño de las tengwar, caracteres con los que se escribirían todas las lenguas élficas de la Tierra Media.

El propósito de este notebook es entrenar un modelo que identifique dichos caracteres tengwar a partir de una fotografía y obtener una traducción, similar a lo que hace Google Lens.

![TN-The_Oath_of_Feanor.jpg](https://tednasmith.poverellomedia.com/wp-content/uploads/2012/07/TN-The_Oath_of_Feanor.jpg)

>Juramento de Fëanor. Ilustrado por Ted Nasmith.

\toc

## Importar librerías y datos

```python
# Librerias
import numpy as np
import pandas as pd
import os
import cv2
from tensorflow import keras
from tensorflow.keras.preprocessing.image import ImageDataGenerator
import kagglehub

# Datos
path = kagglehub.dataset_download("swordey/handwritten-tengwar-letters")
print("Path to dataset files:", path)
```
### Rotar imágenes

Esto con el motivo de ampliar las muestras de nuesto dataset de entenamiento.

```python
# Define the path to your image data directory
data_dir = "/root/.cache/kagglehub/datasets/swordey/handwritten-tengwar-letters/versions/4/tengwar/train"

# Image augmentation configuration
datagen = ImageDataGenerator(
    rotation_range=20,  # Randomly rotate images by up to 20 degrees
    width_shift_range=0.2,  # Randomly shift images horizontally by up to 20% of the image width
    height_shift_range=0.2,  # Randomly shift images vertically by up to 20% of the image height
    shear_range=0.0,  # Apply shearing transformations
    zoom_range=0.2,  # Randomly zoom into images
    horizontal_flip=False, # Randomly flip images horizontally
    fill_mode='nearest'  # Fill in newly created pixels after transformations
)

# Create the image data generator
train_generator = datagen.flow_from_directory(
    data_dir,  # Replace with your actual dataset directory
    target_size=(64, 64),  # Resize images to 64x64
    batch_size=32,
    class_mode='categorical'  # Assuming you have multiple classes
)
```
## Modelo

### Parámetros

```python
# Example model (replace with your own model architecture)
model = keras.Sequential([
    keras.layers.Conv2D(32, (3, 3), activation='relu', input_shape=(64, 64, 1)),
    keras.layers.MaxPooling2D((2, 2)),
    keras.layers.Conv2D(64, (3, 3), activation='relu'),
    keras.layers.MaxPooling2D((2, 2)),
    keras.layers.Flatten(),
    keras.layers.Dense(128, activation='relu'),
    keras.layers.Dense(26, activation='softmax')
])
```
### Compilación

```python
# Compile the model
model.compile(optimizer='adam',
              loss='categorical_crossentropy',
              metrics=['accuracy'])

# Train the model
model.fit(train_generator, epochs=600)  # Adjust the number of epochs as needed

# Save the trained model
model.save("tengwar_model.h5")
```

### Evaluación

```python
# Load the saved model
model = keras.models.load_model("tengwar_model.h5")

# Create a new ImageDataGenerator for the test data (without augmentation)
test_datagen = ImageDataGenerator(rescale=1./255)  # Rescale pixel values

# Create a test generator
test_generator = test_datagen.flow_from_directory(
    # Replace with the path to your test data directory
    "/root/.cache/kagglehub/datasets/swordey/handwritten-tengwar-letters/versions/4/tengwar/test",
    target_size=(64, 64),
    batch_size=32,
    class_mode='categorical',
    shuffle=False  # Important: Do not shuffle the test data
)

# Evaluate the model on the test data
loss, accuracy = model.evaluate(test_generator)
print(f"Test Loss: {loss}")
print(f"Test Accuracy: {accuracy}")

# Get predictions
predictions = model.predict(test_generator)

# Get true labels (assuming your test generator has class_indices)
true_labels = test_generator.classes

# Get class indices mapping
class_indices = test_generator.class_indices

# Invert the class indices dictionary to map indices back to class labels
inverted_class_indices = dict((v,k) for k,v in class_indices.items())

# Convert predicted probabilities to class labels
predicted_labels = np.argmax(predictions, axis=1)

# Convert numerical labels back to original class labels
predicted_labels = [inverted_class_indices[label] for label in predicted_labels]
true_labels = [inverted_class_indices[label] for label in true_labels]

# Compare predictions with true labels
from sklearn.metrics import classification_report, confusion_matrix

print(classification_report(true_labels, predicted_labels))
print(confusion_matrix(true_labels, predicted_labels))
```

### Identificar letras

```python
import string
class_mapping = {i: letter for i, letter in enumerate(string.ascii_uppercase)}

# Load the saved model
model = keras.models.load_model("tengwar_model.h5")

# Function to preprocess the image
def preprocess_image(image_path):
    img = cv2.imread(image_path)
    img = cv2.resize(img, (64, 64))  # Resize to match the model's input size
    img = img / 255.0  # Normalize pixel values
    img = np.expand_dims(img, axis=0)  # Add batch dimension
    return img

# Upload the image
uploaded = files.upload()

for fn in uploaded.keys():
    # Preprocess the uploaded image
    img = preprocess_image(fn)

    # Make a prediction
    prediction = model.predict(img)
    predicted_class = np.argmax(prediction)

    # Assuming you have a mapping from class index to Tengwar letter
    # Replace this with your actual mapping
    class_indices = test_generator.class_indices #class_mapping #list(string.ascii_uppercase)

    predicted_letter = class_indices.get(predicted_class, "Unknown")

    print(f"The predicted Tengwar letter for {fn} is: {predicted_letter}")
```

## Resultados

Malas noticias. No funcionó. Culpo a la mala calidad de los datos de entrenamiento, así como su escasez.
