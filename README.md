# MAGAZINE SOCIAL NETWORK PROJECT
## I. Mô tả dữ liệu
-	Nguồn: https://www.kaggle.com/darinhawley/time-magazine-cover-19232021
-	Mô tả dataset: Dataset hiển thị tất cả những người từng được đăng trên trang nhất của tạp chí Time kể từ khi ra mắt vào năm 1923. trên trang bìa của nó liên quan đến một chủ đề nhất định – ví dụ như trang bìa liên quan đến y học, công nghệ thông tin, khoa học, ….
-	Tập dữ liệu bao gồm 4364 dòng và 6 cột thuộc tính:<br />
<p align="center">
  <img src="https://cdn.discordapp.com/attachments/847349555703316512/1089857131383103608/image.png">
</p>
<!-- ![Screenshot](https://cdn.discordapp.com/attachments/847349555703316512/1089857131383103608/image.png){ style="display: block; margin: 0 auto" } -->
## II.	Tiền xử lý dữ liệu
- Import dữ liệu: Ở đây ta sử dụng cột “Name” là tên của người được đăng trên tạp chí và “Occupation” là thể loại ngành nghề/ lĩnh vực của tạp chí

```python
df = pd.read_csv('/content/timecovers2.csv', usecols = ["Name", "Occupation"])
```

- Loại bỏ các giá trị null

```python
df.dropna() #loại bỏ các giá trị bị thiếu
```

-	Loại bỏ các giá trị trùng lặp

```python
df.drop_duplicates() #loại bỏ các giá trị bị trùng lặp
```
-	Dữ liệu sau khi xử lý
<p align="center">
  <img src="[https://cdn.discordapp.com/attachments/847349555703316512/1089857131383103608/image.png](https://media.discordapp.net/attachments/847349555703316512/1089861357471547462/image.png)">
</p>
