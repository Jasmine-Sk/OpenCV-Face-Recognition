This project was done with this fantastic "Open Source Computer Vision Library", the OpenCV. On this tutorial, we will be focusing on Raspberry Pi (so, Raspbian as OS) and Python, but I also tested the code on My Mac and it also works fine. OpenCV was designed for computational efficiency and with a strong focus on real-time applications. So, it's perfect for real-time face recognition using a camera.

# 3 Phases
To create a complete project on Face Recognition, we must work on 3 very distinct phases:

1. Face Detection and Data Gathering
2. Train the Recognizer
3. Face Recognition
The below block diagram resumes those phases:

![image](https://github.com/Jasmine-Sk/OpenCV-Face-Recognition/assets/126958659/adbcd81e-4df6-409b-83c7-49a6ee72a932)

# Step 1: BoM - Bill of Material
# Main parts:

1. Raspberry Pi V3 - US$ 32.00
2. 5 Megapixels 1080p Sensor OV5647 Mini Camera Video Module - US$ 13.00

# Step 2: Installing OpenCV 3 Package
I am using a Raspberry Pi V3 updated to the last version of Raspbian (Stretch), so the best way to have OpenCV installed, is to follow the excellent tutorial developed by Adrian Rosebrock: Raspbian Stretch: Install OpenCV 3 + Python on your Raspberry Pi.

I tried several different guides to install OpenCV on my Pi. Adrian's tutorial is the best. I advise you to do the same, following his guideline step-by-step.

Once you finished Adrian's tutorial, you should have an OpenCV virtual environment ready to run our experiments on your Pi.

Let's go to our virtual environment and confirm that OpenCV 3 is correctly installed.

Adrian recommends run the command "source" each time you open up a new terminal to ensure your system variables have been set up correctly.

source ~/.profile
Next, let's enter on our virtual environment:

workon cv
If you see the text (cv) preceding your prompt, then you are in the cv virtual environment:

(cv) pi@raspberry:~$
Adrian calls the attention that the cv Python virtual environment is entirely independent and sequestered from the default Python version included in the download of Raspbian Stretch. So, any Python packages in the global site-packages directory will not be available to the cv virtual environment. Similarly, any Python packages installed in site-packages of cv will not be available to the global install of Python.
Now, enter in your Python interpreter:

python
and confirm that you are running the 3.5 (or above) version.

Inside the interpreter (the ">>>" will appear), import the OpenCV library:

import cv2
If no error messages appear, the OpenCV is correctly installed ON YOUR PYTHON VIRTUAL ENVIRONMENT.

You can also check the OpenCV version installed:

cv2.__version__
The 3.3.0 should appear (or a superior version that can be released in future).

![image](https://github.com/Jasmine-Sk/OpenCV-Face-Recognition/assets/126958659/f8f91e40-52f6-4399-960f-5b68d962ef84)

The above Terminal PrintScreen shows the previous steps.

# Step 3: Testing Your Camera
Once you have OpenCV installed in your RPi let's test to confirm that your camera is working properly.

I am assuming that you have a PiCam already installed on your Raspberry Pi.

You must have the camera enabled when you ran through Adrian's tutorial, otherwise, the drivers will not be installed correctly.
In case you get an errar like: OpenCV Error: Assertion failed , you can try solve the issue, using the command:

sudo modprobe bcm2835-v4l2 
Once you have all drivers correctly installed, enter the below Python code on your IDE:

import numpy as np

import cv2

cap = cv2.VideoCapture(0)

cap.set(3,640) # set Width

cap.set(4,480) # set Height

while(True):

    ret, frame = cap.read()
    
    frame = cv2.flip(frame, -1) # Flip camera vertically
    
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    
    cv2.imshow('frame', frame)
    
    cv2.imshow('gray', gray)
    
    k = cv2.waitKey(30) & 0xff
    
    if k == 27: # press 'ESC' to quit
    
        break
cap.release()

cv2.destroyAllWindows()

The above code will capture the video stream that will be generated by your PiCam, displaying both, in BGR color and Gray mode.

Note that I rotated my camera vertically due the way it is assembled. If it is not your case, comment or delete the "flip" command line.
You can alternatively download the code from my GitHub: simpleCamTest.py

To execute, enter the command:
python simpleCamTest.py

To finish the program, you must press the key [ESC] on your keyboard. Click your mouse on the video window, before pressing [ESC].

The above picture shows the result.
Some makers found issues when trying to open the camera ( "Assertion failed" error messages). That could happen if the camera was not enabled during OpenCv installation and so, camera drivers did not install correctly. To correct, use the command:

sudo modprobe bcm2835-v4l2 
You can also add bcm2835-v4l2 to the last line of the /etc/modules file so the driver loads on boot.
To know more about OpenCV, you can follow the tutorial: loading -video-python-opencv-tutorial

# Step 4: Face Detection
The most basic task on Face Recognition is of course, "Face Detecting". Before anything, you must "capture" a face (Phase 1) in order to recognize it, when compared with a new face captured on future (Phase 3).

The most common way to detect a face (or any objects), is using the "Haar Cascade classifier"

Object Detection using Haar feature-based cascade classifiers is an effective object detection method proposed by Paul Viola and Michael Jones in their paper, "Rapid Object Detection using a Boosted Cascade of Simple Features" in 2001. It is a machine learning based approach where a cascade function is trained from a lot of positive and negative images. It is then used to detect objects in other images.

Here we will work with face detection. Initially, the algorithm needs a lot of positive images (images of faces) and negative images (images without faces) to train the classifier. Then we need to extract features from it. The good news is that OpenCV comes with a trainer as well as a detector. If you want to train your own classifier for any object like car, planes etc. you can use OpenCV to create one. Its full details are given here: Cascade Classifier Training.

If you do not want to create your own classifier, OpenCV already contains many pre-trained classifiers for face, eyes, smile, etc. Those XML files can be download from haarcascades directory.

Enough theory, let's create a face detector with OpenCV!

Download the file: faceDetection.py from my GitHub.

import numpy as np

import cv2

faceCascade = cv2.CascadeClassifier('Cascades/haarcascade_frontalface_default.xml')

cap = cv2.VideoCapture(0)

cap.set(3,640) # set Width

cap.set(4,480) # set Height

while True:

    ret, img = cap.read()
    
    img = cv2.flip(img, -1)
    
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    
    faces = faceCascade.detectMultiScale(
    
        gray,     
        
        scaleFactor=1.2,
        
        minNeighbors=5,  
        
        minSize=(20, 20)
        
    )
    
    for (x,y,w,h) in faces:
    
        cv2.rectangle(img,(x,y),(x+w,y+h),(255,0,0),2)
        
        roi_gray = gray[y:y+h, x:x+w]
        
        roi_color = img[y:y+h, x:x+w]  
        
    cv2.imshow('video',img)
    
    k = cv2.waitKey(30) & 0xff
    
    if k == 27: # press 'ESC' to quit
    
        break
        
cap.release()

cv2.destroyAllWindows()

Believe it or not, the above few lines of code are all you need to detect a face, using Python and OpenCV.

When you compare with the last code used to test the camera, you will realize that few parts were added to it. Note the line below:

faceCascade = cv2.CascadeClassifier('Cascades/haarcascade_frontalface_default.xml')
This is the line that loads the "classifier" (that must be in a directory named "Cascades/", under your project directory).

Then, we will set our camera and inside the loop, load our input video in grayscale mode (same we saw before).

Now we must call our classifier function, passing it some very important parameters, as scale factor, number of neighbors and minimum size of the detected face.

faces = faceCascade.detectMultiScale(
        gray,     
        scaleFactor=1.2,
        minNeighbors=5,     
        minSize=(20, 20)
    )
Where,

gray is the input grayscale image.
scaleFactor is the parameter specifying how much the image size is reduced at each image scale. It is used to create the scale pyramid.
minNeighbors is a parameter specifying how many neighbors each candidate rectangle should have, to retain it. A higher number gives lower false positives.
minSize is the minimum rectangle size to be considered a face.
The function will detect faces on the image. Next, we must "mark" the faces in the image, using, for example, a blue rectangle. This is done with this portion of the code:

for (x,y,w,h) in faces:
    cv2.rectangle(img,(x,y),(x+w,y+h),(255,0,0),2)
    roi_gray = gray[y:y+h, x:x+w]
    roi_color = img[y:y+h, x:x+w] 

If faces are found, it returns the positions of detected faces as a rectangle with the left up corner (x,y) and having "w" as its Width and "h" as its Height ==> (x,y,w,h). Please see the picture.
Once we get these locations, we can create an "ROI" (drawn rectangle) for the face and present the result with imshow() function.

# Run the above python Script on your python environment, using the Rpi Terminal:

python faceDetection.py

# Result:

You can also include classifiers for "eyes detection" or even "smile detection". On those cases, you will include the classifier function and rectangle draw inside the face loop, because would be no sense to detect an eye or a smile outside of a face.


