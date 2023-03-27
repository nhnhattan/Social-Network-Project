# MAGAZINE SOCIAL NETWORK PROJECT
## I. Mô tả dữ liệu
-	Nguồn: https://www.kaggle.com/darinhawley/time-magazine-cover-19232021
-	Mô tả dataset: Dataset hiển thị tất cả những người từng được đăng trên trang nhất của tạp chí Time kể từ khi ra mắt vào năm 1923. trên trang bìa của nó liên quan đến một chủ đề nhất định – ví dụ như trang bìa liên quan đến y học, công nghệ thông tin, khoa học, ….
-	Tập dữ liệu bao gồm 4364 dòng và 6 cột thuộc tính:<br />
<p align="center">
  <img src="https://cdn.discordapp.com/attachments/847349555703316512/1089857131383103608/image.png">
</p>
<br />

## II. Tiền xử lý dữ liệu
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
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089861357471547462/image.png">
</p>
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089862405359665192/image.png">
</p>
<br />

## II. Chuyển đổi DataFrame thành đồ thị
### 1. Import các thư viện cần thiết
```python
import pandas as pd
import numpy as np
import matplotlib
import matplotlib.pyplot as plt
import matplotlib.cm as cm
import networkx as nx
from networkx.algorithms import bipartite
from networkx.algorithms import community
import community.community_louvain as community_louvain
from networkx import edge_betweenness_centrality
from random import random
#!pip install scikit-learn
from scipy.spatial.distance import cdist
import sklearn
from sklearn.preprocessing import MinMaxScaler
from sklearn.cluster import KMeans
```
### 2. Đồ thị 2 phía

•	```diff + **Node**```: Phía bên trái là “Occupation” – Lĩnh vực của tạp chí và bên phải là “Name” – Tên của người được đăng trên tạp chí đó
•	**Edge**: Một cạnh tương ứng với khi một người được chọn để đăng tải trên lĩnh vực của tạp chí đó thì cạnh sẽ hình thành. 


























