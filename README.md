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
*	**<ins>Số cụm<ins>**: 17 </br>
o	**<ins>Cụm 0<ins>**: Richard Whitney, Thomas Hart Benton, Salvador Dali </br>
o	**<ins>Cụm 1<ins>**: Dionne Quintuplets, James E. West </br>
o	**<ins>Cụm 2<ins>**: King George V, Emperor Henry Pu Yi, Mohammad Reza Pahlavi, Queen Wilhelmina, Wallis Warfield Simpson, King George VI, King Christian X, Prince Konoye, King Farouk I, King Leopold III </br>
o	**<ins>Cụm 3<ins>**: Chico Marx, Groucho Marx ,Harpo Marx ,Zeppo Marx ,Marie Dressler ,Alice in Wonderland ,Cecil B. DeMille ,Jean Harlow ,Leni Riefenstahl ,Shirley Temple ,Clark Gable ,Marlene Dietrich ,Clyde Beatty ,Billy Mauch ,Bobby Mauch ,Paul Muni </br>
o	**<ins>Cụm 4<ins>**: Lily Pons ,Katharine Cornell ,Lawrence Tibbett ,George M. Cohan ,George Arliss ,Irving Berlin ,Lotte Lehmann ,Richard B. Harrison ,Miriam Hopkins ,Kirsten Flagstad ,Helen Hayes ,Edward Johnson ,Lynn Fontanne, Alfred Lunt </br>
o	**<ins>Cụm 5<ins>**: Sir Arthur Eddington ,Kurt Schmitt ,Dr. Emanuel Libman ,Alexis Carrel, Arthur H. Compton ,Franz Boas ,Joseph DeLee ,Leroy Miner ,Thomas Parran Jr ,Clarence C. Little ,Morris Fishbein ,Ernest O. Lawrence </br>
o	**<ins>Cụm 6<ins>**: Eugene Luther Vidal ,Roscoe Turner ,Edwin C.  </br>
o	**<ins>Cụm 7<ins>**: Elsa Schiaparelli</br> 
o	**<ins>Cụm 8<ins>**: Pope Pius XI ,Charles Coughlin ,Reverend James Perry ,Cardinal Hayes ,Haile Selassie ,Frank N.D. Buchman ,Cardinal Pacelli ,Cardinal Dougherty ,Cosmo Gordon Lang</br>
o	**<ins>Cụm 9<ins>**: Arturo Toscanini ,Jean Sibelius</br>
o	**<ins>Cụm 10<ins>**: Curtis Bok ,Mark Sullivan</br>
o	**<ins>Cụm 11<ins>**: Ellsworth Vines ,Jacob Ruppert ,Jack Crawford ,Edward R. Bradley ,Vernon Gomez ,Cavalcade ,Fred Perry ,Lorne Chabot ,Dizzy Dean ,Donald Budge ,Mickey Cochrane ,Joe DiMaggio ,Helen Hull Jacobs ,Lou Gehrig ,Carl Hubbell ,Bob Feller ,Matt Winn ,Gottfried von Cramm ,Wallace Wade</br>
o	**<ins>Cụm 12<ins>**: Frank Aydelotte, George F. Zook ,Henry N. MacCracken ,Robert M. Hutchins ,Harlow Shapley, James R. Angell</br>
o	**<ins>Cụm 13<ins>**: Edward D. Duffield, Charles F. Kettering, John Hay Whitney, William R. Hearst, Rufus C. Dawes, Juan Trippe, John L. Lewis, Percy S. Straus, Seton Porter, Walter P. Chrysler, Vincent Astor, Errett Lobban Cord, Samuel Insull, Helen Reid, Henry P. Fletcher, Henry Ford, Samuel Clay Williams, John Francis Neylan, Stanley Baldwin, John Cowles, J.P. Morgan, Abby Rockefeller, Marriner S. Eccles, Martin W. Clement, Torkild Rieber, William L. Clayton, Robert McCormick, Joseph M. Patterson, Anthony Grzebyk, William S. Knudsen, Harry B Housser, Colby M. Chester, Walt Disney</br>
o	**<ins>Cụm 14<ins>**: Kurt von Schleicher, Henry L. Stevens Jr, Sadao Araki, Richard Leigh, Italo Balbo, Hermann Goring, Maxime Weygand, Klimentiy Voroshilov, Joseph M. Reeves, Lazaro Cardenas, Gen. Douglas MacArthur, Joseph W. Byrns, Francisco Franco, Mitsumasa Yonai</br>
o	**<ins>Cụm 15<ins>**: T.E. Lawrence, Noël Coward, Gertrude Stein, James Joyce, Thomas C. Mann, Harold Willis Dodds, Louis Barthou, Upton Sinclair, Maxwell Anderson, Kathleen Norris, George Santayana, Isaiah Bowman, John Dos Passos, Virginia Woolf, Sidney Howard, Ernest Hemingway</br>
o	**<ins>Cụm 16<ins>**: Pauline Sabin, Vere Ponsonby Earl of Bessborough, Norman M. Thomas, William G. McAdoo, Yasuya Uchida, Huey P. Long, Rufus D. Isaacs, James Farley, Howard H. Jones, Melvin A. Traylor, Charles Curtis, Norman H. Davis, Henry T. Rainey, Franklin D. Roosevelt, Carter Glass, William W. Atterbury, Pat Harrison, Sara Delano Roosevelt, Adolf Hitler, William H. Woodin, Henry A. Wallace, Cordell Hull, Maxim Litvinov, Raymond Moley, Gerardo Machado, Edouard Daladier, Ferdinand Pecora, Neville Chamberlain, Hugh S. Johnson, Joseph Goebbels, Harold L. Ickes, Frances Perkins, Edward J. Kelly, Engelbert Dollfuss, Fiorello LaGuardia, Joseph V. McKee, John P. O'Brien, George Peek, Eleanor Roosevelt, George F. Warren, Chiang Kai-shek, Elmer Thomas, Jesse H. Jones, James B. Conant, Harry L. Hopkins, Gaston Doumergue, Robert F. Wagner, Robert Lee Doughton, Koki Hirota, Rexford G. Tugwell, Paul von Hindenburg, Joseph B. Poindexter, Donald Richberg, Henry Morgenthau Jr, Joseph C. Grew, Robert M. La Follette Jr, Benjamin N. Cardozo, Herbert H. Lehman, John T. Taylor, Pierre-Etienne Flandin, Frank J. Hogan, Wang Ching-wei, Anthony Eden, Sir John Simon, Harry F. Byrd, Hirosi Saito, John Nance Garner, Joseph Lyons, Joseph T. Robinson, Joseph P. Kennedy Sr, J. Edgar Hoover, Hugo L. Black, Clyde L. Herring, Sir Samuel Hoare, Herbert Hoover, John Buchan - 1st Baron Tweedsmuir, Benito Mussolini, Bruno Mussolini, Vittorio Mussolini, James A. Macauley, Manuel L. Quezon, William Phillips, Alexei Stakhanov, Emperor Hirohito, Joseph Stalin, Emil Hurja, Leon Blum, William E. Borah, Daniel W. Hoan, Alfred Landon, James Allred, Charles P. Taft, Manuel Azana, Emilio Mola, Eugene Talmadge, John D. M. Hamilton, Victor Hope Lord Linlithgow, Edward F. McGrady, George Norris, Leon Trotsky, Thomas E. Dewey, John J. Pelley, Osman Ali Khan, Charles E. Hughes, Joseph E. Davies, Fulgencio Batista, Paul van Zeeland, Ethel du Pont, George Earle III, Harry Bridges, Alben W. Barkley, Mitchell Hepburn, Walter Lippman, William Green, William O. Douglas, Louis D. Brandeis, William B. Bankhead, Madame Chiang Kai-shek
  </br>
  
- Đồ thị Louvain
<p align="center">
  <img src="https://media.discordapp.net/attachments/847349555703316512/1089873517199233095/output.png?width=613&height=434">
</p>

* **Đặc trưng các cụm**:
```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```






















