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
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089869436363755591/image.png">
</p>

- Top 10 người có độ đo Betweeness Centrality cao nhất
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089869436363755591/image.png">
</p>


-	<ins>Ý nghĩa</ins>: Độ đo Betweeness Centrality càng cao thì họ sẽ được đăng trên nhiều lĩnh vực phổ biến với người khác nhất
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089869257258565722/image.png">
</p>
### 2. Closeness Centrality
```python
Closeness_measure = nx.closeness_centrality(G)
sort_Closeness_measure = dict(sorted(Closeness_measure.items(), key=lambda kv:kv[1], reverse =True))
#top5close = pd.DataFrame(list(sort_Closeness_measure.items()), columns=['Node', 'Closeness Measure'])
#display(top5close.head(10))
print("Độ đo Closeness")
display(sort_Closeness_measure)
```
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089869831551078430/image.png?width=546&height=434">
</p>
-	Top 10 người có độ đo Closeness Centrality cao nhất
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089869974224502794/image.png">
</p>

-	<ins>Ý nghĩa</ins>: Độ đo Closeness Centrality càng cao thì họ càng được đăng chung trên nhiều lĩnh vực với những người khác
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089869974509727785/image.png">
</p>

### 3. PageRank
```python
Pagerank =nx.pagerank(G)
sort_Pagerank = dict(sorted(Pagerank.items(), key=lambda kv:kv[1], reverse =True))
#top5pagerank = pd.DataFrame(list(sort_Pagerank.items()), columns=['Node', 'Pagerank'])
#display(top5pagerank.head(10))
print("Độ đo Pagerank")
display(sort_Pagerank)
print(sort_Pagerank)
```
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089870789815312384/image.png?width=605&height=434">
</p>
-	Top 10 người có độ đo PageRank cao nhất
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089870610273939616/image.png">
</p>

-	<ins>Ý nghĩa</ins>: Độ đo PageRank càng cao thì họ càng được đăng nhiều hơn 1 lĩnh vực và có tương tác nhiều với những người được đăng trên những lĩnh vực phổ biến
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089870610529783848/image.png">
</p>

### 4. Clustering Coefficient
```python
clustering_coefficient =nx.clustering(G)
sort_clustering_coefficient = dict(sorted(clustering_coefficient.items(), key=lambda kv:kv[1], reverse =True))
#top5clustering = pd.DataFrame(list(sort_clustering_coefficient.items()), columns=['Node', 'Clustering coefficient'])
#display(top5clustering.head(10))
print("Độ đo Clustering")
display(sort_clustering_coefficient)
```
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089871233262288947/image.png?width=550&height=434">
</p>

-	Top 10 người có độ đo Clustering Coefficient cao nhất
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089871233782386748/image.png">
</p>

-	<ins>Ý nghĩa</ins>: Độ đo Clustering Coefficient càng cao thì họ chỉ được đăng trên chủ yếu 1 lĩnh vực chính
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089871234067595264/image.png">
</p>

## V. Thuật toán gom cụm
### 1. Louvain
```python
#Louvain
plt.figure(figsize=(17, 12), dpi=200) #Vẽ 1 nền trắng với độ phân giải là 200

partition = community_louvain.best_partition(G) #Tính toán sự phân chia tốt nhất

pos = nx.spring_layout(G)#Vẽ đồ thị
cmap = cm.get_cmap('viridis', max(partition.values()) + 1) #Tô màu các node theo cộng đồng của nó
nx.draw_networkx_nodes(G, pos, partition.keys(), node_size=150, cmap=cmap, node_color=list(partition.values()))
nx.draw_networkx_edges(G, pos, alpha=0.5)
nx.draw_networkx_labels(G, pos)
plt.title("Louvain Community")

values = list(partition.values())

print("number of communities: ", len(np.unique(values)))

for i in range(len(np.unique(values))):
  print("nhom", i, "--------------------")
  for name, k in partition.items():
    if k == i:
      print(name)
  print("")
  
plt.show()
```























