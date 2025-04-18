import cv2
from cvzone.HandTrackingModule import HandDetector
from cvzone.ClassificationModule import Classifier
import numpy as np
import math

# Open webcam
cap = cv2.VideoCapture(0)

# Ensure webcam is open
if not cap.isOpened():
    print("Error: Could not open webcam.")
    exit()

# Initialize HandDetector and Classifier
detector = HandDetector(maxHands=1)
classifier = Classifier("D:/converted_keras (2)/keras_model.h5", "D:/converted_keras (2)/labels.txt")

# Constants
offset = 20
imgSize = 300
labels = ["A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O",
          "P", "Q", "R", "S", "T", "U", "V", "X", "Y", "Z"]

while True:
    success, img = cap.read()
    if not success:
        print("Error: Could not read frame from webcam.")
        continue

    imgOutput = img.copy()
    hands, img = detector.findHands(img)

    if hands:
        hand = hands[0]
        x, y, w, h = hand['bbox']

        # Ensure the cropped region is within image bounds
        y1, y2 = max(0, y - offset), min(img.shape[0], y + h + offset)
        x1, x2 = max(0, x - offset), min(img.shape[1], x + w + offset)

        imgCrop = img[y1:y2, x1:x2]

        # Ensure the cropped image has valid size
        if imgCrop.shape[0] == 0 or imgCrop.shape[1] == 0:
            continue

        imgWhite = np.ones((imgSize, imgSize, 3), np.uint8) * 255

        aspectRatio = h / w

        if aspectRatio > 1:
            k = imgSize / h
            wCal = math.ceil(k * w)
            imgResize = cv2.resize(imgCrop, (wCal, imgSize))
            wGap = (imgSize - wCal) // 2
            imgWhite[:, wGap:wGap + wCal] = imgResize
        else:
            k = imgSize / w
            hCal = math.ceil(k * h)
            imgResize = cv2.resize(imgCrop, (imgSize, hCal))
            hGap = (imgSize - hCal) // 2
            imgWhite[hGap:hGap + hCal, :] = imgResize

        # Get prediction and validate index
        prediction, index = classifier.getPrediction(imgWhite, draw=False)

        if 0 <= index < len(labels):  # Ensure valid index
            # Display results
            cv2.rectangle(imgOutput, (x1, y1 - 50), (x1 + 90, y1), (255, 0, 255), cv2.FILLED)
            cv2.putText(imgOutput, labels[index], (x1 + 3, y1 - 7), cv2.FONT_HERSHEY_SIMPLEX,
                        1.5, (255, 255, 255), 2, cv2.LINE_AA)
            cv2.rectangle(imgOutput, (x1, y1), (x2, y2), (255, 0, 255), 4)

        cv2.imshow("ImageCrop", imgCrop)
        cv2.imshow("ImageWhite", imgWhite)

    cv2.imshow("Image", imgOutput)
    if cv2.waitKey(1) & 0xFF == 27:  # Press 'ESC' to exit
        break

cap.release()
cv2.destroyAllWindows()
