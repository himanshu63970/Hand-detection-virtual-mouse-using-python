# Hand-detection-virtual-mouse-using-python

import cv2
import numpy as np
import pyautogui

pyautogui.FAILSAFE = False

cap = cv2.VideoCapture(0)  # 0 for default camera

while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break

    # Convert the frame to HSV for better color detection
    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

    # Define the color range for detecting hands (you may need to adjust these values)
    lower_skin = np.array([0, 20, 70], dtype=np.uint8)
    upper_skin = np.array([20, 255, 255], dtype=np.uint8)

    # Create a mask to filter out everything except skin color
    mask = cv2.inRange(hsv, lower_skin, upper_skin)

    # Perform morphological operations to clean up the mask
    kernel = np.ones((5, 5), np.uint8)
    mask = cv2.erode(mask, kernel, iterations=2)
    mask = cv2.dilate(mask, kernel, iterations=2)

    # Find contours in the mask
    contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

    # Find the largest contour (assuming it's the hand)
    if contours:
        hand_contour = max(contours, key=cv2.contourArea)

        # Get the bounding box of the hand
        x, y, w, h = cv2.boundingRect(hand_contour)

        # Draw the bounding box on the frame
        cv2.rectangle(frame, (x, y), (x + w, y + h), (0, 255, 0), 2)

        # Calculate the center of the bounding box
        cx, cy = x + w // 2, y + h // 2

        # Print bounding box coordinates for debugging
        print("Bounding Box Coordinates:", x, y, x + w, y + h)

        # Move the mouse to the center of the bounding box with adjusted multipliers
        pyautogui.moveTo(cx * 2, cy * 2)  # Adjust the multiplier based on your screen resolution

    # Display the frame
    cv2.imshow('Hand Detection Virtual Mouse', frame)

    # Break the loop when 'q' key is pressed
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
