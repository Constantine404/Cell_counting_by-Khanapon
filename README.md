# Cell_counting_by-Khanapon
# การอธิบายฟังก์ชันที่ใช้ในโปรเจกต์

## 1. โหลดและตรวจสอบภาพ
```python
image = cv2.imread(image_path)
if image is None:
    print("ไม่สามารถโหลดภาพได้ กรุณาตรวจสอบที่อยู่ไฟล์")
    exit()
```
- ใช้ `cv2.imread()` เพื่อโหลดภาพจากไฟล์
- ถ้าภาพโหลดไม่ได้ (`None`), ให้แจ้งเตือนและออกจากโปรแกรม

## 2. แปลงภาพเป็น Grayscale
```python
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
```
- แปลงภาพสีเป็นภาพขาวดำ เพื่อให้การประมวลผลง่ายขึ้น

## 3. ปรับความสว่างและความคมชัด
```python
adjusted = cv2.convertScaleAbs(gray, alpha=1.5, beta=10)
```
- `alpha` ใช้เพิ่มหรือลดความคมชัดของภาพ
- `beta` ใช้เพิ่มหรือลดความสว่างของภาพ

## 4. ปรับ Contrast ด้วย CLAHE
```python
clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8, 8))
enhanced = clahe.apply(adjusted)
```
- ใช้เทคนิค CLAHE (Contrast Limited Adaptive Histogram Equalization) เพื่อเพิ่ม contrast เฉพาะจุด

## 5. ลดสัญญาณรบกวนด้วย Gaussian Blur
```python
blurred = cv2.GaussianBlur(enhanced, (1, 1), 0)
```
- ใช้ Gaussian Blur เพื่อทำให้ภาพดูนุ่มนวลและลด noise ที่ไม่จำเป็น

## 6. ใช้ Threshold เพื่อสร้างภาพขาวดำ (Binary Image)
```python
_, binary = cv2.threshold(blurred, 90, 255, cv2.THRESH_BINARY_INV)
```
- เปลี่ยนภาพ grayscale ให้เป็นขาวดำ โดยกำหนดค่า threshold เป็น 90
- ใช้ `cv2.THRESH_BINARY_INV` ทำให้บริเวณที่มืดกลายเป็นสีขาวและสว่างกลายเป็นสีดำ

## 7. กำจัดสัญญาณรบกวนด้วย Morphological Operations
```python
kernel = np.ones((3,3), np.uint8)
opening = cv2.morphologyEx(binary, cv2.MORPH_OPEN, kernel, iterations=1)
```
- ใช้ `cv2.MORPH_OPEN` เพื่อลบ noise เล็ก ๆ ออกจากภาพ binary

## 8. หา Contours ของจุดสีเข้ม
```python
contours, _ = cv2.findContours(opening, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
```
- ค้นหา contours ในภาพที่ผ่านการ threshold
- ใช้ `cv2.RETR_EXTERNAL` เพื่อหาเฉพาะ contour ด้านนอกสุด

## 9. คัดกรอง Contours ตามขนาด
```python
filtered_contours = []
for contour in contours:
    area = cv2.contourArea(contour)
    if min_area < area < max_area:
        filtered_contours.append(contour)
```
- ใช้ `cv2.contourArea()` วัดขนาดของ contour
- กรองเอาเฉพาะจุดที่มีขนาดเหมาะสม (ไม่น้อยเกินไปและไม่ใหญ่เกินไป)

## 10. คำนวณจุดศูนย์กลางของ Contours
```python
centroids = []
for contour in filtered_contours:
    M = cv2.moments(contour)
    if M["m00"] > 0:
        cX = int(M["m10"] / M["m00"])
        cY = int(M["m01"] / M["m00"])
        centroids.append((cX, cY))
```
- คำนวณจุดศูนย์กลางของ contour แต่ละอันโดยใช้ `cv2.moments()`

## 11. วาดวงกลมรอบเซลล์และใส่หมายเลข
```python
for i, (cX, cY) in enumerate(centroids):
    cv2.circle(result, (cX, cY), 15, (0, 255, 0), 2)
    cv2.putText(result, str(i+1), (cX-10, cY), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255, 0, 0), 2)
```
- ใช้ `cv2.circle()` วาดวงกลมรอบเซลล์ที่ตรวจพบ
- ใช้ `cv2.putText()` ใส่หมายเลขกำกับแต่ละเซลล์

## 12. แสดงผลลัพธ์และบันทึกภาพ
```python
cv2.imshow("Original", image)
cv2.imshow("Enhanced", enhanced)
cv2.imshow("Binary", opening)
cv2.imshow("Detected Cells", result)
cv2.waitKey(0)
cv2.destroyAllWindows()
```
- ใช้ `cv2.imshow()` แสดงภาพแต่ละขั้นตอน
- ใช้ `cv2.waitKey(0)` เพื่อรอให้ผู้ใช้กดปุ่มก่อนปิดหน้าต่าง

```python
cv2.imwrite("detected_cells_result.jpg", result)
```
- บันทึกภาพที่มีการวาดวงกลมรอบเซลล์ที่ตรวจพบ

---

## 🔹 สรุป
โค้ดนี้ใช้สำหรับตรวจจับเซลล์ในภาพโดยใช้ OpenCV โดยมีขั้นตอนหลัก ๆ คือ:
1. โหลดภาพและแปลงเป็น grayscale
2. ปรับความสว่างและ contrast
3. กำจัดสัญญาณรบกวน
4. แปลงภาพเป็น binary และใช้ morphological operations
5. ค้นหา contours และคัดกรองขนาด
6. คำนวณจุดศูนย์กลางและวาดวงกลมรอบเซลล์
7. แสดงและบันทึกผลลัพธ์

---
🎯 **หากต้องการปรับค่า threshold หรือ parameter อื่น ๆ สามารถทดลองเปลี่ยนค่าต่าง ๆ เช่น `alpha`, `beta`, `clipLimit` และ `min_area` ได้!**
