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

•	**Node**: Phía bên trái là “Occupation” – Lĩnh vực của tạp chí và bên phải là “Name” – Tên của người được đăng trên tạp chí đó
•	**Edge**: Một cạnh tương ứng với khi một người được chọn để đăng tải trên lĩnh vực của tạp chí đó thì cạnh sẽ hình thành. 

```python
print("Số thể loại tạp chí ngành nghề", Occupation.nunique())
print("Số người được đăng ", Name.nunique())
print("Số cạnh ", len(df))
```
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089862405359665192/image.png">
</p>
- Có thể thấy ta có tất cả 17 ngành nghề và số lượng người mẫu được đăng tải trên tạp chí là 280. Cuối cùng là số cạnh được hình thành là 300.
* **Đồ thị 2 phía**

```python
B = nx.Graph()
Occupation = df['Occupation']
Name = df['Name']

for index, row in df.iterrows():
    B.add_edge(row['Occupation'], row['Name'], weight=1)
B.add_nodes_from(Name, bipartite = 0) #thêm nút source thuộc tập hợp 0
B.add_nodes_from(Occupation, bipartite = 1) #thêm nút source thuộc tập hợp 1

plt.figure(figsize=(12, 12)) #Tạo một hình trắng mới với size 20,20
pos = nx.spring_layout(B) #Vẽ đồ thị
#plt.subplots()là một hàm trả về một bộ giá trị chứa (các) đối tượng hình và trục
#Vì vậy, khi sử dụng, fig, ax = plt.subplots() ta giải nén bộ này vào các biến fig và ax.
fig, ax = plt.subplots(1,1, figsize=(12,12), dpi = 200) #dpi là độ phân giải
nx.draw_networkx(B, pos = nx.drawing.layout.bipartite_layout(B,Occupation),font_size=8,width=0.4) #draw_networkx(B: đồ thị, pos: khóa và vị trí)
```
<p align="center">
  <img src="https://cdn.discordapp.com/attachments/847349555703316512/1089866895487926333/output.png">
</p>
-	Nhìn vào đồ thị, ta thấy rằng một lĩnh vực có thể có nhiều người được đăng khác nhau và một người có thể được đăng trên nhiều lĩnh vực khác nhau

### 3. Đồ thị 1 phía
•	**Node**: Là tên của người được đăng trên tạp chí đó</br>
•	**Edge**: Hình thành khi 2 người được đăng trên cùng 1 lĩnh vực tạp chí, Ý nghĩa cho ta thấy sự cạnh tranh để giữ được vị trí của mình tại lĩnh vực đó </br>
<ins>Ví dụ</ins>: Như 2 người cùng được đăng trên 1 lĩnh vực là Politics & Gov thì họ sẽ được nối thành 1 cạnh
•	**Weight**: Trọng số là số lĩnh vực trùng nhau mà 2 người cùng được đăng trên lĩnh vực đó 

```python
G = bipartite.weighted_projected_graph(B, Name) #Trả về một phép chiếu có trọng số của B lên một trong các tập hợp node của nó.

plt.figure(num=None, figsize=(17, 12), dpi=100, facecolor='w') #Tạo 1 hình trắng

layout = nx.spring_layout(G)

nx.draw_networkx_nodes( 
    G,
    layout,
    nodelist = Name,
    node_size =100,
    node_color = 'blue'
)

nx.draw_networkx_edges(G, layout, edge_color="k", width=0.3) #Vẽ cạnh với màu là xám #cccccc
node_labels = dict(zip(Name, Name)) #dict và zip để nối các node quen cùng 1 người với nhau
nx.draw_networkx_labels(G, layout, labels=node_labels) 
plt.axis('off') #Tắt trục, xóa trục x và y khỏi biểu đồ

plt.title("Graph Team") #Tên đồ thị

plt.show() #in đồ thị
```
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089868094584922122/output.png?width=612&height=434">
</p>

## IV. Các độ đo
### 1. Betweeness Centrality
```python
betweenness_centrality = nx.betweenness_centrality(G)
sort_betweenness_centrality = dict(sorted(betweenness_centrality.items(), key=lambda kv:kv[1], reverse =True))
#top5between = pd.DataFrame(list(sort_betweenness_centrality.items()), columns=['Node', 'Betweenness Centrality'])
#display(top5between.head(10))
print("Độ đo Betweeness")
print()
display(sort_betweenness_centrality)
```
- Độ đo Betweeness Centrality được xếp theo tên
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089868947064639499/image.png?width=624&height=434">
</p>

















