# Face Recognition Attendance System

This project implements a face recognition-based attendance system using Python, OpenCV, and the `face_recognition` library. It uses a webcam feed to recognize faces from a set of pre-trained images, and records the attendance of the recognized individuals in a CSV file with timestamps.

## Table of Contents
1. [Installation](#installation)
2. [Usage](#usage)
3. [Code Explanation](#code-explanation)
   - [Face Encoding](#face-encoding)
   - [Marking Attendance](#marking-attendance)
   - [Webcam Face Recognition](#webcam-face-recognition)
4. [Troubleshooting](#troubleshooting)
5. [License](#license)
6. [Acknowledgments](#acknowledgments)

## Installation

### Prerequisites

Ensure you have Python 3.x installed on your system. You will also need to install the following dependencies:

- `opencv-python`
- `numpy`
- `face_recognition`

You can install these dependencies using pip:

```bash
pip install opencv-python numpy face_recognition
```

### Cloning the Repository

Clone this repository using the following command:

```bash
git clone <repository-url>
cd <repository-directory>
```

Replace \`<repository-url>\` with the actual URL of your GitHub repository.

## Usage

### Preparing Training Images

1. **Training Images:**
   - Create a folder called `Training_images` in your project directory.
   - Place the images of individuals you want to recognize in this folder. Make sure each image file is named after the person it represents (e.g., `John.jpg`, `Jane.jpg`).

2. **Run the Script:**
   Run the Python script to start the face recognition-based attendance system:

   ```bash
   python main.py
   ```

### Attendance Recording

- The recognized individuals' names, along with the time of recognition, are recorded in a CSV file called `Attendance.csv`.
- If a person is already marked as present, they will not be marked again during the same session.

## Code Explanation

### Face Encoding

1. **Loading Training Images:**
   The script reads the images from the `Training_images` directory and converts them into face encodings using the `face_recognition` library.

   ```python
   path = 'Training_images'
   images = []
   classNames = []
   myList = os.listdir(path)

   for cl in myList:
       curImg = cv2.imread(f'{path}/{cl}')
       images.append(curImg)
       classNames.append(os.path.splitext(cl)[0])
   ```

2. **Finding Encodings:**
   The `findEncodings` function converts each image into a list of encodings. These encodings are later used to compare with live video frames to recognize faces.

   ```python
   def findEncodings(images):
       encodeList = []
       for img in images:
           img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
           encode = face_recognition.face_encodings(img)[0]
           encodeList.append(encode)
       return encodeList
   ```

### Marking Attendance

- **Marking Attendance in CSV:** When a face is recognized, the name is recorded in `Attendance.csv` along with the time of recognition. The system ensures that a person is not marked more than once in a session.

```python
def markAttendance(name):
    with open('Attendance.csv', 'r+') as f:
        myDataList = f.readlines()
        nameList = []
        for line in myDataList:
            entry = line.split(',')
            nameList.append(entry[0])
        if name not in nameList:
            now = datetime.now()
            dtString = now.strftime('%H:%M:%S')
            f.writelines(f'\n{name},{dtString}')
```

### Webcam Face Recognition

1. **Capturing Video:** The script uses `cv2.VideoCapture(0)` to capture live video from the webcam.

2. **Face Detection and Recognition:**
   - The script processes the video feed in real-time, detects faces, and compares them with the pre-encoded face data.
   - If a match is found, the person's name is displayed on the video frame and recorded in the attendance CSV.

```python
while True:
    success, img = cap.read()
    imgS = cv2.resize(img, (0, 0), None, 0.25, 0.25)
    imgS = cv2.cvtColor(imgS, cv2.COLOR_BGR2RGB)

    facesCurFrame = face_recognition.face_locations(imgS)
    encodesCurFrame = face_recognition.face_encodings(imgS, facesCurFrame)

    for encodeFace, faceLoc in zip(encodesCurFrame, facesCurFrame):
        matches = face_recognition.compare_faces(encodeListKnown, encodeFace)
        faceDis = face_recognition.face_distance(encodeListKnown, encodeFace)
        matchIndex = np.argmin(faceDis)

        if matches[matchIndex]:
            name = classNames[matchIndex].upper()
            y1, x2, y2, x1 = faceLoc
            y1, x2, y2, x1 = y1 * 4, x2 * 4, y2 * 4, x1 * 4
            cv2.rectangle(img, (x1, y1), (x2, y2), (0, 255, 0), 2)
            cv2.rectangle(img, (x1, y2 - 35), (x2, y2), (0, 255, 0), cv2.FILLED)
            cv2.putText(img, name, (x1 + 6, y2 - 6), cv2.FONT_HERSHEY_COMPLEX, 1, (255, 255, 255), 2)
            markAttendance(name)

    cv2.imshow('Webcam', img)
    cv2.waitKey(1)
```

## Troubleshooting

1. **Error: \`list index out of range\`**
   - This error occurs if no face is detected in the training images. Ensure that all the images in the `Training_images` folder contain clear, recognizable faces.
   - You can add a check before accessing `face_recognition.face_encodings(img)[0]` to ensure that faces are detected in the image.

2. **Webcam Not Opening**
   - Ensure that your webcam is properly connected and that `cv2.VideoCapture(0)` is using the correct webcam index.
   - If you have multiple cameras, you may need to adjust the index (e.g., `cv2.VideoCapture(1)`).

3. **No Face Detected in Video**
   - Ensure the video is clear and well-lit. Poor lighting can affect the ability of the model to detect faces.
   - Test with a variety of angles and distances to make sure the face is within the frame.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

This project uses the following open-source libraries:
- OpenCV
- face_recognition
- NumPy
