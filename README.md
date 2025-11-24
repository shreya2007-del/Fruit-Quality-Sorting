import cv2
import numpy as np

# MERGE SORT (Descending order)
def merge_sort(arr):
    if len(arr) > 1:
        mid = len(arr)//2
        L = arr[:mid]
        R = arr[mid:]
        merge_sort(L)
        merge_sort(R)
        i = j = k = 0
        while i < len(L) and j < len(R):
            if L[i][1] > R[j][1]:
                arr[k] = L[i]
                i += 1
            else:
                arr[k] = R[j]
                j += 1
            k += 1
        while i < len(L):
            arr[k] = L[i]
            i += 1
            k += 1

        while j < len(R):
            arr[k] = R[j]
            j += 1
            k += 1

# FUNCTION: Quality Message Based on Score
def quality_message(score):
    if score >= 70:
        return "BEST QUALITY"
    elif score >= 50:
        return "GOOD QUALITY"
    elif score >= 30:
        return "AVERAGE QUALITY"
    else:
        return "POOR QUALITY"

# FUNCTION: Calculate Fruit Quality Score
def get_quality_score(image_path):
    img = cv2.imread(image_path)
    if img is None:
        print(f"Image not found: {image_path}")
        return 0
    img = cv2.resize(img, (300, 300))
    hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
    # Ripeness indicator (Saturation)
    mean_saturation = np.mean(hsv[:, :, 1])
    # Threshold for segmentation
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    _, th = cv2.threshold(gray, 0, 255, cv2.THRESH_OTSU)
    # Detect contours
    contours, _ = cv2.findContours(th, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    if len(contours) == 0:
        print("No fruit detected!")
        return 0
    cnt = max(contours, key=cv2.contourArea)
    area = cv2.contourArea(cnt)
    perimeter = cv2.arcLength(cnt, True)
    if perimeter == 0:
        circularity = 0
    else:
        circularity = (4 * np.pi * area) / (perimeter ** 2)

    # FINAL SCORE (Weights: 40 + 60)
    score = (mean_saturation / 255 * 40) + (circularity * 60)
    return score

# FRUIT LIST (Add your own image names)
fruit_images = ["apple1.jpg", "apple2.jpg", "apple3.jpg","apple4.jpg",]
fruits = []
for img in fruit_images:
    score = get_quality_score(img)
    fruits.append((img, score))

# SORT FRUITS BY QUALITY
merge_sort(fruits)
print("\n==== SORTED FRUITS (BEST TO WORST) ====")
for fruit, score in fruits:
    print(f"{fruit} - Score: {score:.2f} - {quality_message(score)}")
