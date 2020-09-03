### Source
https://johnricco.github.io/2017/04/04/python-html/


```python
import os
import requests
import urllib
import math
import copy
import pandas as pd	
import numpy as np
from bs4 import BeautifulSoup 

class html_tables(object):
    
    def __init__(self, url):
        
        self.url      = url
        self.r        = requests.get(self.url)
        self.url_soup = BeautifulSoup(self.r.text)
        
    def read(self):
        
        self.tables      = []
        self.tables_html = self.url_soup.find_all("table")
        
        # Parse each table
        for n in range(0, len(self.tables_html)):
            
            n_cols = 0
            n_rows = 0
            
            for row in self.tables_html[n].find_all("tr"):
                col_tags = row.find_all(["td", "th"])
                if len(col_tags) > 0:
                    n_rows += 1
                    if len(col_tags) > n_cols:
                        n_cols = len(col_tags)
            
            # Create dataframe
            df = pd.DataFrame(index = range(0, n_rows), columns = range(0, n_cols))
            
            # Create list to store rowspan values 
            skip_index = [0 for i in range(0, n_cols)]
            
            # Start by iterating over each row in this table...
            row_counter = 0
            for row in self.tables_html[n].find_all("tr"):
                
                # Skip row if it's blank
                if len(row.find_all(["td", "th"])) == 0:
                    next
                
                else:
                    
                    # Get all cells containing data in this row
                    columns = row.find_all(["td", "th"])
                    col_dim = []
                    row_dim = []
                    col_dim_counter = -1
                    row_dim_counter = -1
                    col_counter = -1
                    this_skip_index = copy.deepcopy(skip_index)
                    
                    for col in columns:
                        
                        # Determine cell dimensions
                        colspan = col.get("colspan")
                        if colspan is None:
                            col_dim.append(1)
                        else:
                            col_dim.append(int(colspan))
                        col_dim_counter += 1
                            
                        rowspan = col.get("rowspan")
                        if rowspan is None:
                            row_dim.append(1)
                        else:
                            row_dim.append(int(rowspan))
                        row_dim_counter += 1
                            
                        # Adjust column counter
                        if col_counter == -1:
                            col_counter = 0  
                        else:
                            col_counter = col_counter + col_dim[col_dim_counter - 1]
                            
                        while skip_index[col_counter] > 0:
                            col_counter += 1

                        # Get cell contents  
                        cell_data = col.get_text()
                        
                        # Insert data into cell
                        df.iat[row_counter, col_counter] = cell_data

                        # Record column skipping index
                        if row_dim[row_dim_counter] > 1:
                            this_skip_index[col_counter] = row_dim[row_dim_counter]
                
                # Adjust row counter 
                row_counter += 1
                
                # Adjust column skipping index
                skip_index = [i - 1 if i > 0 else i for i in this_skip_index]

            # Append dataframe to list of tables
            self.tables.append(df)
        
        return(self.tables)
```


```python
ssa_url = "https://vi.wikipedia.org/wiki/B%E1%BA%A3n_m%E1%BA%ABu:B%E1%BA%A3ng_th%C3%B4ng_tin_COVID-19_t%E1%BA%A1i_Vi%E1%BB%87t_Nam"
ssa = html_tables(ssa_url)
first_table = ssa.read()[0]
first_table.to_csv("ssa.csv", header = False, index = False)
```

### Cleaning


```python
df = pd.read_csv('ssa.csv')
```


```python
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>BN\n</th>
      <th>Thời gian  (2020)\n</th>
      <th>Tuổi\n</th>
      <th>Giới tính\n</th>
      <th>Nơi xác nhận nhiễm\n</th>
      <th>Quốc tịch\n</th>
      <th>Bệnh viện điều trị[gc 1]\n</th>
      <th>Từng ở Trung Quốc\n</th>
      <th>Từng ở các quốc gia khác[gc 2]\n</th>
      <th>Tình trạng[gc 3]\n</th>
      <th>Ghi chú\n</th>
      <th>Nguồn\n</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01\n</td>
      <td>23/01\n</td>
      <td>66\n</td>
      <td>Nam\n</td>
      <td>TP. Hồ Chí Minh\n</td>
      <td>Trung Quốc\n</td>
      <td>Bệnh viện Chợ Rẫy\n</td>
      <td>Có\n</td>
      <td>Không\n</td>
      <td>Đã xuất viện\n</td>
      <td>Đi cùng vợ từ Vũ Hán sang Việt Nam thăm #2 (co...</td>
      <td>[1]\n</td>
    </tr>
    <tr>
      <th>1</th>
      <td>02\n</td>
      <td>NaN</td>
      <td>28\n</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Có\n</td>
      <td>Không\n</td>
      <td>Đã xuất viện\n</td>
      <td>Con #1\n</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>03\n</td>
      <td>30/01\n</td>
      <td>25\n</td>
      <td>Nữ\n</td>
      <td>Thanh Hóa\n</td>
      <td>Việt Nam\n</td>
      <td>Bệnh viện Đa khoa Thanh Hóa\n</td>
      <td>Có\n</td>
      <td>Không\n</td>
      <td>Đã xuất viện\n</td>
      <td>Tập huấn từ Vũ Hán về.\n</td>
      <td>[1]\n</td>
    </tr>
    <tr>
      <th>3</th>
      <td>04\n</td>
      <td>NaN</td>
      <td>29\n</td>
      <td>Nam\n</td>
      <td>Vĩnh Phúc\n</td>
      <td>NaN</td>
      <td>Bệnh viện Bệnh nhiệt đới Trung ương\n</td>
      <td>Có\n</td>
      <td>Không\n</td>
      <td>Đã xuất viện\n</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>05\n</td>
      <td>NaN</td>
      <td>23\n</td>
      <td>Nữ\n</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Có\n</td>
      <td>Không\n</td>
      <td>Đã xuất viện\n</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
new_columns = [column.strip() for column in df.columns]
df.columns = new_columns
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>BN</th>
      <th>Thời gian  (2020)</th>
      <th>Tuổi</th>
      <th>Giới tính</th>
      <th>Nơi xác nhận nhiễm</th>
      <th>Quốc tịch</th>
      <th>Bệnh viện điều trị[gc 1]</th>
      <th>Từng ở Trung Quốc</th>
      <th>Từng ở các quốc gia khác[gc 2]</th>
      <th>Tình trạng[gc 3]</th>
      <th>Ghi chú</th>
      <th>Nguồn</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01\n</td>
      <td>23/01\n</td>
      <td>66\n</td>
      <td>Nam\n</td>
      <td>TP. Hồ Chí Minh\n</td>
      <td>Trung Quốc\n</td>
      <td>Bệnh viện Chợ Rẫy\n</td>
      <td>Có\n</td>
      <td>Không\n</td>
      <td>Đã xuất viện\n</td>
      <td>Đi cùng vợ từ Vũ Hán sang Việt Nam thăm #2 (co...</td>
      <td>[1]\n</td>
    </tr>
    <tr>
      <th>1</th>
      <td>02\n</td>
      <td>NaN</td>
      <td>28\n</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Có\n</td>
      <td>Không\n</td>
      <td>Đã xuất viện\n</td>
      <td>Con #1\n</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>03\n</td>
      <td>30/01\n</td>
      <td>25\n</td>
      <td>Nữ\n</td>
      <td>Thanh Hóa\n</td>
      <td>Việt Nam\n</td>
      <td>Bệnh viện Đa khoa Thanh Hóa\n</td>
      <td>Có\n</td>
      <td>Không\n</td>
      <td>Đã xuất viện\n</td>
      <td>Tập huấn từ Vũ Hán về.\n</td>
      <td>[1]\n</td>
    </tr>
    <tr>
      <th>3</th>
      <td>04\n</td>
      <td>NaN</td>
      <td>29\n</td>
      <td>Nam\n</td>
      <td>Vĩnh Phúc\n</td>
      <td>NaN</td>
      <td>Bệnh viện Bệnh nhiệt đới Trung ương\n</td>
      <td>Có\n</td>
      <td>Không\n</td>
      <td>Đã xuất viện\n</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>05\n</td>
      <td>NaN</td>
      <td>23\n</td>
      <td>Nữ\n</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Có\n</td>
      <td>Không\n</td>
      <td>Đã xuất viện\n</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
df = df.replace('\n','', regex=True)
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>BN</th>
      <th>Thời gian  (2020)</th>
      <th>Tuổi</th>
      <th>Giới tính</th>
      <th>Nơi xác nhận nhiễm</th>
      <th>Quốc tịch</th>
      <th>Bệnh viện điều trị[gc 1]</th>
      <th>Từng ở Trung Quốc</th>
      <th>Từng ở các quốc gia khác[gc 2]</th>
      <th>Tình trạng[gc 3]</th>
      <th>Ghi chú</th>
      <th>Nguồn</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01</td>
      <td>23/01</td>
      <td>66</td>
      <td>Nam</td>
      <td>TP. Hồ Chí Minh</td>
      <td>Trung Quốc</td>
      <td>Bệnh viện Chợ Rẫy</td>
      <td>Có</td>
      <td>Không</td>
      <td>Đã xuất viện</td>
      <td>Đi cùng vợ từ Vũ Hán sang Việt Nam thăm #2 (co...</td>
      <td>[1]</td>
    </tr>
    <tr>
      <th>1</th>
      <td>02</td>
      <td>NaN</td>
      <td>28</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Có</td>
      <td>Không</td>
      <td>Đã xuất viện</td>
      <td>Con #1</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>03</td>
      <td>30/01</td>
      <td>25</td>
      <td>Nữ</td>
      <td>Thanh Hóa</td>
      <td>Việt Nam</td>
      <td>Bệnh viện Đa khoa Thanh Hóa</td>
      <td>Có</td>
      <td>Không</td>
      <td>Đã xuất viện</td>
      <td>Tập huấn từ Vũ Hán về.</td>
      <td>[1]</td>
    </tr>
    <tr>
      <th>3</th>
      <td>04</td>
      <td>NaN</td>
      <td>29</td>
      <td>Nam</td>
      <td>Vĩnh Phúc</td>
      <td>NaN</td>
      <td>Bệnh viện Bệnh nhiệt đới Trung ương</td>
      <td>Có</td>
      <td>Không</td>
      <td>Đã xuất viện</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>05</td>
      <td>NaN</td>
      <td>23</td>
      <td>Nữ</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Có</td>
      <td>Không</td>
      <td>Đã xuất viện</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
df = df.fillna(method='ffill')
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>BN</th>
      <th>Thời gian  (2020)</th>
      <th>Tuổi</th>
      <th>Giới tính</th>
      <th>Nơi xác nhận nhiễm</th>
      <th>Quốc tịch</th>
      <th>Bệnh viện điều trị[gc 1]</th>
      <th>Từng ở Trung Quốc</th>
      <th>Từng ở các quốc gia khác[gc 2]</th>
      <th>Tình trạng[gc 3]</th>
      <th>Ghi chú</th>
      <th>Nguồn</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01</td>
      <td>23/01</td>
      <td>66</td>
      <td>Nam</td>
      <td>TP. Hồ Chí Minh</td>
      <td>Trung Quốc</td>
      <td>Bệnh viện Chợ Rẫy</td>
      <td>Có</td>
      <td>Không</td>
      <td>Đã xuất viện</td>
      <td>Đi cùng vợ từ Vũ Hán sang Việt Nam thăm #2 (co...</td>
      <td>[1]</td>
    </tr>
    <tr>
      <th>1</th>
      <td>02</td>
      <td>23/01</td>
      <td>28</td>
      <td>Nam</td>
      <td>TP. Hồ Chí Minh</td>
      <td>Trung Quốc</td>
      <td>Bệnh viện Chợ Rẫy</td>
      <td>Có</td>
      <td>Không</td>
      <td>Đã xuất viện</td>
      <td>Con #1</td>
      <td>[1]</td>
    </tr>
    <tr>
      <th>2</th>
      <td>03</td>
      <td>30/01</td>
      <td>25</td>
      <td>Nữ</td>
      <td>Thanh Hóa</td>
      <td>Việt Nam</td>
      <td>Bệnh viện Đa khoa Thanh Hóa</td>
      <td>Có</td>
      <td>Không</td>
      <td>Đã xuất viện</td>
      <td>Tập huấn từ Vũ Hán về.</td>
      <td>[1]</td>
    </tr>
    <tr>
      <th>3</th>
      <td>04</td>
      <td>30/01</td>
      <td>29</td>
      <td>Nam</td>
      <td>Vĩnh Phúc</td>
      <td>Việt Nam</td>
      <td>Bệnh viện Bệnh nhiệt đới Trung ương</td>
      <td>Có</td>
      <td>Không</td>
      <td>Đã xuất viện</td>
      <td>Tập huấn từ Vũ Hán về.</td>
      <td>[1]</td>
    </tr>
    <tr>
      <th>4</th>
      <td>05</td>
      <td>30/01</td>
      <td>23</td>
      <td>Nữ</td>
      <td>Vĩnh Phúc</td>
      <td>Việt Nam</td>
      <td>Bệnh viện Bệnh nhiệt đới Trung ương</td>
      <td>Có</td>
      <td>Không</td>
      <td>Đã xuất viện</td>
      <td>Tập huấn từ Vũ Hán về.</td>
      <td>[1]</td>
    </tr>
  </tbody>
</table>
</div>




```python
df = df.rename(columns={'Bệnh viện điều trị[gc 1]': 'Bệnh viện điều trị',
                        'Từng ở các quốc gia khác[gc 2]': 'Từng ở các quốc gia khác', 
                       'Tình trạng[gc 3]': 'Tình trạng'})
```


```python
df = df.drop(axis=1, labels=['Nguồn'])
```


```python
len(df)
```




    1045




```python
df.iloc[1044]
```




    BN                           Tính đến 18 giờ 00 phút ngày 31 tháng 8 năm 2020
    Thời gian  (2020)                                                       31/08
    Tuổi                                                                       62
    Giới tính                                                                 Nam
    Nơi xác nhận nhiễm                                                    Phú Thọ
    Quốc tịch                                                               Ấn Độ
    Bệnh viện điều trị                        Bệnh viện Bệnh nhiệt đới Trung ương
    Từng ở Trung Quốc                                                       Không
    Từng ở các quốc gia khác                                                   Có
    Tình trạng                                                      Đang điều trị
    Ghi chú                     Hành khách trên chuyến bay 6E8679 từ Ấn Độ về ...
    Name: 1044, dtype: object




```python
df = df.drop(axis=0, labels=1044)
```


```python
df.to_excel('covid_vietnam.xlsx', index = False)
```

# --------------------------


```python
import pandas as pd
```


```python
data = '["Trung Quốc--BN001","-BN001-BN002","-BN001-BN003","Trung Quốc--BN004","Trung Quốc--BN005","Trung Quốc--BN006","Trung Quốc--BN007","Trung Quốc--BN008","Trung Quốc--BN009","-BN006-BN010","-BN006-BN011","-BN006-BN012","-BN006-BN013","Trung Quốc--BN014","-BN010-BN015","-BN006-BN016","Anh--BN017","Hàn Quốc--BN018","-BN017-BN019","-BN017-BN020","Anh--BN021","Anh--BN022","Anh--BN023","Anh--BN024","Anh--BN025","Anh--BN026","Anh--BN027","Anh--BN028","Anh--BN029","Anh--BN030","Anh--BN031","Anh--BN032","Anh--BN033","Mỹ--BN034","-BN029-BN035","-BN034-BN036","-BN034-BN037","-BN034-BN038","-BN034-BN039","-BN034-BN040","-BN034-BN041","-BN034-BN042","-BN034-BN043","-BN038-BN044","-BN034-BN045","Anh--BN046","-BN017-BN047","-BN034-BN048","-BN030-BN049","Anh--BN050","Pháp--BN051","Qatar--BN052","Qatar--BN053","TBN--BN054","Anh--BN055","Pháp--BN056","Anh--BN057","Pháp--BN058","Anh--BN059","Pháp--BN060","Malaysia--BN061","Anh--BN062","Anh--BN063","Thụy Sĩ--BN064","-BN045-BN065","Mỹ--BN066","Malaysia--BN067","Singapore--BN068","Nga--BN069","Anh--BN070","Anh--BN071","-BN060-BN072","Anh--BN073","Pháp--BN074","Anh--BN075","Pháp--BN076","Anh--BN077","Anh--BN078","Anh--BN079","Anh--BN080","Pháp--BN081","Anh--BN082","Thổ Nhĩ Kỳ--BN083","Anh--BN084","Anh--BN085","--BN086","-BN086-BN087","Anh--BN088","Mỹ--BN089","TBN--BN090","--BN091","Pháp--BN092","Hungary--BN093","Séc--BN094","Pháp--BN095","Pháp--BN096","--BN097","--BN098","Pháp--BN099","Malaysia--BN100","Anh--BN101","Anh--BN102","Anh--BN103","Anh--BN104","Malaysia--BN105","Malaysia--BN106","-BN086-BN107","Anh--BN108","Anh--BN109","Mỹ--BN110","Anh--BN111","Pháp--BN112","Anh--BN113","Hà Lan--BN114","Séc--BN115","-BN028-BN116","Campuchia--BN117","Campuchia--BN118","--BN119","--BN120","Mỹ--BN121","Thái Lan--BN122","Malaysia--BN123","--BN124","--BN125","--BN126","--BN127","Anh--BN128","Anh--BN129","TBN--BN130","TBN--BN131","TBN--BN132","--BN133","Nga--BN134","Đan Mạch--BN135","Mỹ--BN136","Đức--BN137","Anh--BN138","Anh--BN139","Anh--BN140","-BN028-BN141","Mỹ--BN142","Mỹ--BN143","Anh--BN144","Anh--BN145","Thái Lan--BN146","Anh--BN147","Pháp--BN148","Đức--BN149","Mỹ--BN150","-BN124-BN151","-BN127-BN152","Úc--BN153","Anh--BN154","Anh--BN155","Anh--BN156","--BN157","--BN158","--BN159","TBN--BN160","--BN161","--BN162","--BN163","Anh--BN164","Anh--BN165","Thái Lan--BN166","Đan Mạch--BN167","--BN168","--BN169","--BN170","Mỹ--BN171","--BN172","Nga--BN173","--BN174","--BN175","--BN176","--BN177","--BN178","Ả Rập--BN179","Pháp--BN180","Thái Lan--BN181","Thụy sĩ--BN182","-BN148-BN183","--BN184","--BN185","Thổ Nhĩ Kỳ-BN076-BN186","Mỹ--BN187","-BN169-BN188","--BN189","--BN190","--BN191","--BN192","--BN193","--BN194","--BN195","--BN196","--BN197","--BN198","--BN199","--BN200","--BN201","--BN202","Thổ Nhĩ Kỳ--BN203","Séc--BN204","--BN205","-BN124-BN206","-BN124-BN207","--BN208","-BN163-BN209","Thái Lan--BN210","Mỹ--BN211","Nga--BN212","--BN213","--BN214","--BN215","Đức--BN216","Nhật Bản--BN217","Nga--BN218","--BN219","Pháp--BN220","Canada--BN221","Mỹ--BN222","--BN223","--BN224","Nga--BN225","Nga--BN226","-BN209-BN227","--BN228","--BN229","--BN230","--BN231","Nga--BN232","Nga--BN233","Pháp--BN234","--BN235","--BN236","--BN237","Thái Lan--BN238","--BN239","Thái Lan--BN240","Anh--BN241","Nga--BN242","--BN243","Đức--BN244","TBN--BN245","Nga--BN246","-BN124-BN247","Mỹ--BN248","Mỹ--BN249","-BN243-BN250","--BN251","Campuchia--BN252","-BN243-BN253","-BN243-BN254","Nga--BN255","Nga--BN256","-BN243-BN257","-BN243-BN258","-BN243-BN259","-BN243-BN260","--BN261","-BN254-BN262","--BN263","--BN264","Nga--BN265","--BN266","-BN243-BN267","--BN268","Nhật Bản--BN269","Nhật Bản--BN270","Anh--BN271","UAE--BN272","UAE--BN273","UAE--BN274","UAE--BN275","UAE--BN276","UAE--BN277","UAE--BN278","UAE--BN279","UAE--BN280","UAE--BN281","UAE--BN282","UAE--BN283","UAE--BN284","UAE--BN285","UAE--BN286","UAE--BN287","UAE--BN288","Nga--BN289","Nga--BN290","Nga--BN291","Nga--BN292","Nga--BN293","Nga--BN294","Nga--BN295","Nga--BN296","Nga--BN297","Nga--BN298","Nga--BN299","Nga--BN300","Nga--BN301","Nga--BN302","Nga--BN303","Nga--BN304","Nga--BN305","Nga--BN306","Nga--BN307","Nga--BN308","Nga--BN309","Nga--BN310","Nga--BN311","Nga--BN312","UAE--BN313","Nga--BN314","Campuchia--BN315","Philippines--BN316","Nga--BN317","Nga--BN318","Nga--BN319","Nga--BN320","Nga--BN321","Nga--BN322","Mỹ--BN323","Mỹ--BN324","Nga--BN325","Pháp--BN326","Nga--BN327","Nga--BN328","Anh--BN329","Mexico--BN330","Mexico--BN331","Campuchia--BN332","Malaysia--BN333","Trung Quốc--BN334","Kuwait--BN335","Kuwait--BN336","Kuwait--BN337","Kuwait--BN338","Kuwait--BN339","Kuwait--BN340","Kuwait--BN341","Kuwait--BN342","Thụy Điển--BN343","Thụy Điển--BN344","Thụy Điển--BN345","Thụy Điển--BN346","Thụy Điển--BN347","Thụy Điển--BN348","Thụy Điển--BN349","Kuwait--BN350","Kuwait--BN351","Kuwait--BN352","Cameroon--BN353","Kuwait--BN354","Kuwait--BN355","Bangladesh--BN356","Bangladesh--BN357","Bangladesh--BN358","Bangladesh--BN359","Bangladesh--BN360","Bangladesh--BN361","Bangladesh--BN362","Bangladesh--BN363","Bangladesh--BN364","Bangladesh--BN365","Bangladesh--BN366","Bangladesh--BN367","Bangladesh--BN368","Bangladesh--BN369","Oman--BN370","Nga--BN371","Nga--BN372","Nga--BN373","Nga--BN374","Nga--BN375","Nga--BN376","Nga--BN377","Nga--BN378","Nga--BN379","Nga--BN380","Nga--BN381","Nga--BN382","Nhật Bản--BN383","Nga--BN384","Nga--BN385","Nga--BN386","Nga--BN387","Nga--BN388","Nga--BN389","Nga--BN390","Nga--BN391","Nga--BN392","Nga--BN393","Nga--BN394","Nga--BN395","Nga--BN396","Nga--BN397","Nga--BN398","Nga--BN399","Nga--BN400","Nga--BN401","Nga--BN402","Nga--BN403","Nga--BN404","Nga--BN405","Nga--BN406","Nga--BN407","Nga--BN408","Hàn Quốc--BN409","Nga--BN410","Nga--BN411","Nga--BN412","Nhật Bản--BN413","Nga--BN414","Nga--BN415","--BN416","Nga--BN417","--BN418","--BN419","--BN420","--BN421","--BN422","--BN423","--BN424","--BN425","--BN426","--BN427","--BN428","--BN429","--BN430","--BN431","--BN432","--BN433","--BN434","--BN435","--BN436","--BN437","--BN438","--BN439","--BN440","--BN441","--BN442","--BN443","--BN444","--BN445","--BN446","--BN447","--BN448","--BN449","--BN450","--BN451","--BN452","--BN453","--BN454","--BN455","--BN456","--BN457","--BN458","--BN459","--BN460","--BN461","--BN462","--BN463","--BN464","--BN465","--BN466","--BN467","--BN468","--BN469","--BN470","--BN471","--BN472","--BN473","--BN474","--BN475","--BN476","--BN477","--BN478","--BN479","--BN480","--BN481","--BN482","--BN483","--BN484","--BN485","--BN486","--BN487","--BN488","--BN489","--BN490","--BN491","--BN492","--BN493","--BN494","--BN495","--BN496","--BN497","--BN498","--BN499","--BN500","--BN501","--BN502","--BN503","--BN504","--BN505","--BN506","--BN507","--BN508","--BN509","--BN510","--BN511","--BN512","--BN513","--BN514","--BN515","--BN516","--BN517","--BN518","--BN519","--BN520","--BN521","--BN522","--BN523","--BN524","--BN525","--BN526","Guinea-Bissau--BN527","Guinea-Bissau--BN528","Guinea-Bissau--BN529","Guinea-Bissau--BN530","Guinea-Bissau--BN531","Guinea-Bissau--BN532","Guinea-Bissau--BN533","Guinea-Bissau--BN534","Guinea-Bissau--BN535","Guinea-Bissau--BN536","Guinea-Bissau--BN537","Guinea-Bissau--BN538","Guinea-Bissau--BN539","Guinea-Bissau--BN540","Guinea-Bissau--BN541","Guinea-Bissau--BN542","Guinea-Bissau--BN543","Guinea-Bissau--BN544","Guinea-Bissau--BN545","Guinea-Bissau--BN546","--BN547","--BN548","-BN461-BN549","-BN461-BN550","-BN461-BN551","--BN552","--BN553","-BN416-BN554","--BN555","--BN556","-BN509-BN557","--BN558","Indonesia--BN559","Indonesia--BN560","--BN561","--BN562","--BN563","--BN564","--BN565","-BN522-BN566","--BN567","--BN568","--BN569","--BN570","--BN571","--BN572","--BN573","--BN574","--BN575","--BN576","--BN577","--BN578","--BN579","--BN580","--BN581","--BN582","--BN583","--BN584","--BN585","--BN586","Nga--BN587","Nga--BN588","--BN589","-BN517-BN590","--BN591","--BN592","--BN593","--BN594","--BN595","--BN596","--BN597","--BN598","--BN599","--BN600","--BN601","--BN602","Mỹ--BN603","--BN604","--BN605","--BN606","--BN607","-BN434-BN608","--BN609","--BN610","--BN611","-BN480-BN612","--BN613","--BN614","--BN615","--BN616","--BN617","--BN618","--BN619","--BN620","--BN621","--BN622","--BN623","--BN624","--BN625","-BN524-BN626","-BN525-BN627","--BN628","-BN509-BN629","-BN509-BN630","-BN510-BN631","-BN488-BN632","-BN488-BN633","-BN510-BN634","-BN466-BN635","-BN456-BN636","-BN456-BN637","-BN456-BN638","-BN501-BN639","-BN456-BN640","-BN427-BN641","--BN642","--BN643","-BN555-BN644","-BN555-BN645","--BN646","--BN647","--BN648","--BN649","--BN650","--BN651","--BN652","--BN653","--BN654","-BN473-BN655","--BN656","-BN510-BN657","--BN658","--BN659","--BN660","--BN661","--BN662","-BN503-BN663","-BN420-BN664","-BN448-BN665","--BN666","-BN628-BN667","-BN628-BN668","-BN510-BN669","--BN670","-BN469-BN671"]'
```


```python
data
```




    '["Trung Quốc--BN001","-BN001-BN002","-BN001-BN003","Trung Quốc--BN004","Trung Quốc--BN005","Trung Quốc--BN006","Trung Quốc--BN007","Trung Quốc--BN008","Trung Quốc--BN009","-BN006-BN010","-BN006-BN011","-BN006-BN012","-BN006-BN013","Trung Quốc--BN014","-BN010-BN015","-BN006-BN016","Anh--BN017","Hàn Quốc--BN018","-BN017-BN019","-BN017-BN020","Anh--BN021","Anh--BN022","Anh--BN023","Anh--BN024","Anh--BN025","Anh--BN026","Anh--BN027","Anh--BN028","Anh--BN029","Anh--BN030","Anh--BN031","Anh--BN032","Anh--BN033","Mỹ--BN034","-BN029-BN035","-BN034-BN036","-BN034-BN037","-BN034-BN038","-BN034-BN039","-BN034-BN040","-BN034-BN041","-BN034-BN042","-BN034-BN043","-BN038-BN044","-BN034-BN045","Anh--BN046","-BN017-BN047","-BN034-BN048","-BN030-BN049","Anh--BN050","Pháp--BN051","Qatar--BN052","Qatar--BN053","TBN--BN054","Anh--BN055","Pháp--BN056","Anh--BN057","Pháp--BN058","Anh--BN059","Pháp--BN060","Malaysia--BN061","Anh--BN062","Anh--BN063","Thụy Sĩ--BN064","-BN045-BN065","Mỹ--BN066","Malaysia--BN067","Singapore--BN068","Nga--BN069","Anh--BN070","Anh--BN071","-BN060-BN072","Anh--BN073","Pháp--BN074","Anh--BN075","Pháp--BN076","Anh--BN077","Anh--BN078","Anh--BN079","Anh--BN080","Pháp--BN081","Anh--BN082","Thổ Nhĩ Kỳ--BN083","Anh--BN084","Anh--BN085","--BN086","-BN086-BN087","Anh--BN088","Mỹ--BN089","TBN--BN090","--BN091","Pháp--BN092","Hungary--BN093","Séc--BN094","Pháp--BN095","Pháp--BN096","--BN097","--BN098","Pháp--BN099","Malaysia--BN100","Anh--BN101","Anh--BN102","Anh--BN103","Anh--BN104","Malaysia--BN105","Malaysia--BN106","-BN086-BN107","Anh--BN108","Anh--BN109","Mỹ--BN110","Anh--BN111","Pháp--BN112","Anh--BN113","Hà Lan--BN114","Séc--BN115","-BN028-BN116","Campuchia--BN117","Campuchia--BN118","--BN119","--BN120","Mỹ--BN121","Thái Lan--BN122","Malaysia--BN123","--BN124","--BN125","--BN126","--BN127","Anh--BN128","Anh--BN129","TBN--BN130","TBN--BN131","TBN--BN132","--BN133","Nga--BN134","Đan Mạch--BN135","Mỹ--BN136","Đức--BN137","Anh--BN138","Anh--BN139","Anh--BN140","-BN028-BN141","Mỹ--BN142","Mỹ--BN143","Anh--BN144","Anh--BN145","Thái Lan--BN146","Anh--BN147","Pháp--BN148","Đức--BN149","Mỹ--BN150","-BN124-BN151","-BN127-BN152","Úc--BN153","Anh--BN154","Anh--BN155","Anh--BN156","--BN157","--BN158","--BN159","TBN--BN160","--BN161","--BN162","--BN163","Anh--BN164","Anh--BN165","Thái Lan--BN166","Đan Mạch--BN167","--BN168","--BN169","--BN170","Mỹ--BN171","--BN172","Nga--BN173","--BN174","--BN175","--BN176","--BN177","--BN178","Ả Rập--BN179","Pháp--BN180","Thái Lan--BN181","Thụy sĩ--BN182","-BN148-BN183","--BN184","--BN185","Thổ Nhĩ Kỳ-BN076-BN186","Mỹ--BN187","-BN169-BN188","--BN189","--BN190","--BN191","--BN192","--BN193","--BN194","--BN195","--BN196","--BN197","--BN198","--BN199","--BN200","--BN201","--BN202","Thổ Nhĩ Kỳ--BN203","Séc--BN204","--BN205","-BN124-BN206","-BN124-BN207","--BN208","-BN163-BN209","Thái Lan--BN210","Mỹ--BN211","Nga--BN212","--BN213","--BN214","--BN215","Đức--BN216","Nhật Bản--BN217","Nga--BN218","--BN219","Pháp--BN220","Canada--BN221","Mỹ--BN222","--BN223","--BN224","Nga--BN225","Nga--BN226","-BN209-BN227","--BN228","--BN229","--BN230","--BN231","Nga--BN232","Nga--BN233","Pháp--BN234","--BN235","--BN236","--BN237","Thái Lan--BN238","--BN239","Thái Lan--BN240","Anh--BN241","Nga--BN242","--BN243","Đức--BN244","TBN--BN245","Nga--BN246","-BN124-BN247","Mỹ--BN248","Mỹ--BN249","-BN243-BN250","--BN251","Campuchia--BN252","-BN243-BN253","-BN243-BN254","Nga--BN255","Nga--BN256","-BN243-BN257","-BN243-BN258","-BN243-BN259","-BN243-BN260","--BN261","-BN254-BN262","--BN263","--BN264","Nga--BN265","--BN266","-BN243-BN267","--BN268","Nhật Bản--BN269","Nhật Bản--BN270","Anh--BN271","UAE--BN272","UAE--BN273","UAE--BN274","UAE--BN275","UAE--BN276","UAE--BN277","UAE--BN278","UAE--BN279","UAE--BN280","UAE--BN281","UAE--BN282","UAE--BN283","UAE--BN284","UAE--BN285","UAE--BN286","UAE--BN287","UAE--BN288","Nga--BN289","Nga--BN290","Nga--BN291","Nga--BN292","Nga--BN293","Nga--BN294","Nga--BN295","Nga--BN296","Nga--BN297","Nga--BN298","Nga--BN299","Nga--BN300","Nga--BN301","Nga--BN302","Nga--BN303","Nga--BN304","Nga--BN305","Nga--BN306","Nga--BN307","Nga--BN308","Nga--BN309","Nga--BN310","Nga--BN311","Nga--BN312","UAE--BN313","Nga--BN314","Campuchia--BN315","Philippines--BN316","Nga--BN317","Nga--BN318","Nga--BN319","Nga--BN320","Nga--BN321","Nga--BN322","Mỹ--BN323","Mỹ--BN324","Nga--BN325","Pháp--BN326","Nga--BN327","Nga--BN328","Anh--BN329","Mexico--BN330","Mexico--BN331","Campuchia--BN332","Malaysia--BN333","Trung Quốc--BN334","Kuwait--BN335","Kuwait--BN336","Kuwait--BN337","Kuwait--BN338","Kuwait--BN339","Kuwait--BN340","Kuwait--BN341","Kuwait--BN342","Thụy Điển--BN343","Thụy Điển--BN344","Thụy Điển--BN345","Thụy Điển--BN346","Thụy Điển--BN347","Thụy Điển--BN348","Thụy Điển--BN349","Kuwait--BN350","Kuwait--BN351","Kuwait--BN352","Cameroon--BN353","Kuwait--BN354","Kuwait--BN355","Bangladesh--BN356","Bangladesh--BN357","Bangladesh--BN358","Bangladesh--BN359","Bangladesh--BN360","Bangladesh--BN361","Bangladesh--BN362","Bangladesh--BN363","Bangladesh--BN364","Bangladesh--BN365","Bangladesh--BN366","Bangladesh--BN367","Bangladesh--BN368","Bangladesh--BN369","Oman--BN370","Nga--BN371","Nga--BN372","Nga--BN373","Nga--BN374","Nga--BN375","Nga--BN376","Nga--BN377","Nga--BN378","Nga--BN379","Nga--BN380","Nga--BN381","Nga--BN382","Nhật Bản--BN383","Nga--BN384","Nga--BN385","Nga--BN386","Nga--BN387","Nga--BN388","Nga--BN389","Nga--BN390","Nga--BN391","Nga--BN392","Nga--BN393","Nga--BN394","Nga--BN395","Nga--BN396","Nga--BN397","Nga--BN398","Nga--BN399","Nga--BN400","Nga--BN401","Nga--BN402","Nga--BN403","Nga--BN404","Nga--BN405","Nga--BN406","Nga--BN407","Nga--BN408","Hàn Quốc--BN409","Nga--BN410","Nga--BN411","Nga--BN412","Nhật Bản--BN413","Nga--BN414","Nga--BN415","--BN416","Nga--BN417","--BN418","--BN419","--BN420","--BN421","--BN422","--BN423","--BN424","--BN425","--BN426","--BN427","--BN428","--BN429","--BN430","--BN431","--BN432","--BN433","--BN434","--BN435","--BN436","--BN437","--BN438","--BN439","--BN440","--BN441","--BN442","--BN443","--BN444","--BN445","--BN446","--BN447","--BN448","--BN449","--BN450","--BN451","--BN452","--BN453","--BN454","--BN455","--BN456","--BN457","--BN458","--BN459","--BN460","--BN461","--BN462","--BN463","--BN464","--BN465","--BN466","--BN467","--BN468","--BN469","--BN470","--BN471","--BN472","--BN473","--BN474","--BN475","--BN476","--BN477","--BN478","--BN479","--BN480","--BN481","--BN482","--BN483","--BN484","--BN485","--BN486","--BN487","--BN488","--BN489","--BN490","--BN491","--BN492","--BN493","--BN494","--BN495","--BN496","--BN497","--BN498","--BN499","--BN500","--BN501","--BN502","--BN503","--BN504","--BN505","--BN506","--BN507","--BN508","--BN509","--BN510","--BN511","--BN512","--BN513","--BN514","--BN515","--BN516","--BN517","--BN518","--BN519","--BN520","--BN521","--BN522","--BN523","--BN524","--BN525","--BN526","Guinea-Bissau--BN527","Guinea-Bissau--BN528","Guinea-Bissau--BN529","Guinea-Bissau--BN530","Guinea-Bissau--BN531","Guinea-Bissau--BN532","Guinea-Bissau--BN533","Guinea-Bissau--BN534","Guinea-Bissau--BN535","Guinea-Bissau--BN536","Guinea-Bissau--BN537","Guinea-Bissau--BN538","Guinea-Bissau--BN539","Guinea-Bissau--BN540","Guinea-Bissau--BN541","Guinea-Bissau--BN542","Guinea-Bissau--BN543","Guinea-Bissau--BN544","Guinea-Bissau--BN545","Guinea-Bissau--BN546","--BN547","--BN548","-BN461-BN549","-BN461-BN550","-BN461-BN551","--BN552","--BN553","-BN416-BN554","--BN555","--BN556","-BN509-BN557","--BN558","Indonesia--BN559","Indonesia--BN560","--BN561","--BN562","--BN563","--BN564","--BN565","-BN522-BN566","--BN567","--BN568","--BN569","--BN570","--BN571","--BN572","--BN573","--BN574","--BN575","--BN576","--BN577","--BN578","--BN579","--BN580","--BN581","--BN582","--BN583","--BN584","--BN585","--BN586","Nga--BN587","Nga--BN588","--BN589","-BN517-BN590","--BN591","--BN592","--BN593","--BN594","--BN595","--BN596","--BN597","--BN598","--BN599","--BN600","--BN601","--BN602","Mỹ--BN603","--BN604","--BN605","--BN606","--BN607","-BN434-BN608","--BN609","--BN610","--BN611","-BN480-BN612","--BN613","--BN614","--BN615","--BN616","--BN617","--BN618","--BN619","--BN620","--BN621","--BN622","--BN623","--BN624","--BN625","-BN524-BN626","-BN525-BN627","--BN628","-BN509-BN629","-BN509-BN630","-BN510-BN631","-BN488-BN632","-BN488-BN633","-BN510-BN634","-BN466-BN635","-BN456-BN636","-BN456-BN637","-BN456-BN638","-BN501-BN639","-BN456-BN640","-BN427-BN641","--BN642","--BN643","-BN555-BN644","-BN555-BN645","--BN646","--BN647","--BN648","--BN649","--BN650","--BN651","--BN652","--BN653","--BN654","-BN473-BN655","--BN656","-BN510-BN657","--BN658","--BN659","--BN660","--BN661","--BN662","-BN503-BN663","-BN420-BN664","-BN448-BN665","--BN666","-BN628-BN667","-BN628-BN668","-BN510-BN669","--BN670","-BN469-BN671"]'




```python
type(data)
```




    str




```python
import re
```


```python
# https://stackoverflow.com/questions/171480/regex-grabbing-values-between-quotation-marks
cells = re.findall(r'"(.*?)"', data)
```


```python
len(cells)
```




    671




```python
df = pd.DataFrame(cells, columns=['text'])
```


```python
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Trung Quốc--BN001</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-BN001-BN002</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-BN001-BN003</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Trung Quốc--BN004</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Trung Quốc--BN005</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>666</th>
      <td>-BN628-BN667</td>
    </tr>
    <tr>
      <th>667</th>
      <td>-BN628-BN668</td>
    </tr>
    <tr>
      <th>668</th>
      <td>-BN510-BN669</td>
    </tr>
    <tr>
      <th>669</th>
      <td>--BN670</td>
    </tr>
    <tr>
      <th>670</th>
      <td>-BN469-BN671</td>
    </tr>
  </tbody>
</table>
<p>671 rows × 1 columns</p>
</div>



### F0


```python
# https://www.regexpal.com/115897
```


```python
f0_rule = r'((?P<Country>[a-zA-ZÀÁÂÃÈÉÊÌÍÒÓÔÕÙÚĂĐĨŨƠàáâãèéêìíòóôõùúăđĩũơƯĂẠẢẤẦẨẪẬẮẰẲẴẶẸẺẼỀỀỂẾưăạảấầẩẫậắằẳẵặẹẻẽềềểếỄỆỈỊỌỎỐỒỔỖỘỚỜỞỠỢỤỦỨỪễệỉịọỏốồổỗộớờởỡợụủứừỬỮỰỲỴÝỶỸửữựỳỵỷỹ]+(?:\s)?(?:-)?(?:[a-zA-ZÀÁÂÃÈÉÊÌÍÒÓÔÕÙÚĂĐĨŨƠàáâãèéêìíòóôõùúăđĩũơƯĂẠẢẤẦẨẪẬẮẰẲẴẶẸẺẼỀỀỂẾưăạảấầẩẫậắằẳẵặẹẻẽềềểếỄỆỈỊỌỎỐỒỔỖỘỚỜỞỠỢỤỦỨỪễệỉịọỏốồổỗộớờởỡợụủứừỬỮỰỲỴÝỶỸửữựỳỵỷỹ]+)?)--(?P<Number>BN\d{3}))'
```


```python
f0 = df['text'].str.extract(f0_rule)
```


```python
f0['Country'].unique()
```




    array(['Trung Quốc', nan, 'Anh', 'Hàn Quốc', 'Mỹ', 'p', 'Qatar', 'TBN',
           'Malaysia', 'Thụy Sĩ', 'Singapore', 'Nga', 'Hungary', 'c', 'Lan',
           'Campuchia', 'Thái Lan', 'Đan Mạch', 'Úc', 'Ả Rập', 'Thụy sĩ',
           'Nhật Bản', 'Canada', 'UAE', 'Philippines', 'Mexico', 'Kuwait',
           'Thụy Điển', 'Cameroon', 'Bangladesh', 'Oman', 'Guinea-Bissau',
           'Indonesia'], dtype=object)




```python
f0_valid = f0[f0['Country'].notnull()]
```


```python
f0_valid['Country'].unique()
```




    array(['Trung Quốc', 'Anh', 'Hàn Quốc', 'Mỹ', 'p', 'Qatar', 'TBN',
           'Malaysia', 'Thụy Sĩ', 'Singapore', 'Nga', 'Hungary', 'c', 'Lan',
           'Campuchia', 'Thái Lan', 'Đan Mạch', 'Úc', 'Ả Rập', 'Thụy sĩ',
           'Nhật Bản', 'Canada', 'UAE', 'Philippines', 'Mexico', 'Kuwait',
           'Thụy Điển', 'Cameroon', 'Bangladesh', 'Oman', 'Guinea-Bissau',
           'Indonesia'], dtype=object)




```python
# Dropping France
f0_dropped = f0_valid.drop(f0_valid[f0_valid['Country'] == 'p'].index, inplace = False)
# Dropping Czech
f0_dropped = f0_dropped.drop(f0_valid[f0_valid['Country'] == 'c'].index, inplace = False)
# Dropping Netherlands
f0_dropped = f0_dropped.drop(f0_valid[f0_valid['Country'] == 'Lan'].index, inplace = False)
# Fixing Switzerland
f0_dropped = f0_dropped.replace(to_replace = 'Thụy sĩ', value='Thụy Sĩ')

f0_dropped['Country'].unique()
```




    array(['Trung Quốc', 'Anh', 'Hàn Quốc', 'Mỹ', 'Qatar', 'TBN', 'Malaysia',
           'Thụy Sĩ', 'Singapore', 'Nga', 'Hungary', 'Campuchia', 'Thái Lan',
           'Đan Mạch', 'Úc', 'Ả Rập', 'Nhật Bản', 'Canada', 'UAE',
           'Philippines', 'Mexico', 'Kuwait', 'Thụy Điển', 'Cameroon',
           'Bangladesh', 'Oman', 'Guinea-Bissau', 'Indonesia'], dtype=object)



Trung Quốc
Anh
Hàn Quốc
Mỹ
Pháp
Qatar
TBN
Malaysia
Thụy Sĩ
Singapore
Nga
**Thổ Nhĩ Kỳ**
Hungary
**Séc**
Hà Lan
Campuchia
Thái Lan
Đan Mạch
Đức
Úc
Ả Rập
Thụy Sĩ
Nhật Bản
Canada
UAE
Philippines
Mexico
Kuwait
Thụy Điển
Cameroon
Bangladesh
Oman
Guinea Bissau
Indonesia


```python
f0_dropped
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>Country</th>
      <th>Number</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Trung Quốc--BN001</td>
      <td>Trung Quốc</td>
      <td>BN001</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Trung Quốc--BN004</td>
      <td>Trung Quốc</td>
      <td>BN004</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Trung Quốc--BN005</td>
      <td>Trung Quốc</td>
      <td>BN005</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Trung Quốc--BN006</td>
      <td>Trung Quốc</td>
      <td>BN006</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Trung Quốc--BN007</td>
      <td>Trung Quốc</td>
      <td>BN007</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>558</th>
      <td>Indonesia--BN559</td>
      <td>Indonesia</td>
      <td>BN559</td>
    </tr>
    <tr>
      <th>559</th>
      <td>Indonesia--BN560</td>
      <td>Indonesia</td>
      <td>BN560</td>
    </tr>
    <tr>
      <th>586</th>
      <td>Nga--BN587</td>
      <td>Nga</td>
      <td>BN587</td>
    </tr>
    <tr>
      <th>587</th>
      <td>Nga--BN588</td>
      <td>Nga</td>
      <td>BN588</td>
    </tr>
    <tr>
      <th>602</th>
      <td>Mỹ--BN603</td>
      <td>Mỹ</td>
      <td>BN603</td>
    </tr>
  </tbody>
</table>
<p>300 rows × 3 columns</p>
</div>




```python
france = r'((?P<Country>Pháp)--(?P<Number>BN\d{3}))'
france_df_all = df['text'].str.extract(france)
france_df_all['Country'].unique()

france_df = france_df_all[france_df_all['Country'] == 'Pháp']
print(len(france_df))
france_df
```

    17
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>Country</th>
      <th>Number</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>50</th>
      <td>Pháp--BN051</td>
      <td>Pháp</td>
      <td>BN051</td>
    </tr>
    <tr>
      <th>55</th>
      <td>Pháp--BN056</td>
      <td>Pháp</td>
      <td>BN056</td>
    </tr>
    <tr>
      <th>57</th>
      <td>Pháp--BN058</td>
      <td>Pháp</td>
      <td>BN058</td>
    </tr>
    <tr>
      <th>59</th>
      <td>Pháp--BN060</td>
      <td>Pháp</td>
      <td>BN060</td>
    </tr>
    <tr>
      <th>73</th>
      <td>Pháp--BN074</td>
      <td>Pháp</td>
      <td>BN074</td>
    </tr>
    <tr>
      <th>75</th>
      <td>Pháp--BN076</td>
      <td>Pháp</td>
      <td>BN076</td>
    </tr>
    <tr>
      <th>80</th>
      <td>Pháp--BN081</td>
      <td>Pháp</td>
      <td>BN081</td>
    </tr>
    <tr>
      <th>91</th>
      <td>Pháp--BN092</td>
      <td>Pháp</td>
      <td>BN092</td>
    </tr>
    <tr>
      <th>94</th>
      <td>Pháp--BN095</td>
      <td>Pháp</td>
      <td>BN095</td>
    </tr>
    <tr>
      <th>95</th>
      <td>Pháp--BN096</td>
      <td>Pháp</td>
      <td>BN096</td>
    </tr>
    <tr>
      <th>98</th>
      <td>Pháp--BN099</td>
      <td>Pháp</td>
      <td>BN099</td>
    </tr>
    <tr>
      <th>111</th>
      <td>Pháp--BN112</td>
      <td>Pháp</td>
      <td>BN112</td>
    </tr>
    <tr>
      <th>147</th>
      <td>Pháp--BN148</td>
      <td>Pháp</td>
      <td>BN148</td>
    </tr>
    <tr>
      <th>179</th>
      <td>Pháp--BN180</td>
      <td>Pháp</td>
      <td>BN180</td>
    </tr>
    <tr>
      <th>219</th>
      <td>Pháp--BN220</td>
      <td>Pháp</td>
      <td>BN220</td>
    </tr>
    <tr>
      <th>233</th>
      <td>Pháp--BN234</td>
      <td>Pháp</td>
      <td>BN234</td>
    </tr>
    <tr>
      <th>325</th>
      <td>Pháp--BN326</td>
      <td>Pháp</td>
      <td>BN326</td>
    </tr>
  </tbody>
</table>
</div>




```python
# germany = r'(?P<Germany>Đức)'
germany = r'((?P<Country>Đức)--(?P<Number>BN\d{3}))'
germany_df_all = df['text'].str.extract(germany)
germany_df_all['Country'].unique()

# germany_df = germany_df_all[germany_df_all['Germany'] == 'Đức']
germany_df = germany_df_all[germany_df_all['Country'].notnull()]
germany_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>Country</th>
      <th>Number</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>136</th>
      <td>Đức--BN137</td>
      <td>Đức</td>
      <td>BN137</td>
    </tr>
    <tr>
      <th>148</th>
      <td>Đức--BN149</td>
      <td>Đức</td>
      <td>BN149</td>
    </tr>
    <tr>
      <th>215</th>
      <td>Đức--BN216</td>
      <td>Đức</td>
      <td>BN216</td>
    </tr>
    <tr>
      <th>243</th>
      <td>Đức--BN244</td>
      <td>Đức</td>
      <td>BN244</td>
    </tr>
  </tbody>
</table>
</div>




```python
netherlands = r'((?P<Country>Hà Lan)--(?P<Number>BN\d{3}))'
netherlands_df_all = df['text'].str.extract(netherlands)
netherlands_df_all['Country'].unique()

# netherlands_df = netherlands_df_all[netherlands_df_all['Netherlands'] == 'Hà Lan']
netherlands_df = netherlands_df_all[netherlands_df_all['Country'].notnull()]
netherlands_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>Country</th>
      <th>Number</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>113</th>
      <td>Hà Lan--BN114</td>
      <td>Hà Lan</td>
      <td>BN114</td>
    </tr>
  </tbody>
</table>
</div>




```python
czech = r'((?P<Country>Séc)--(?P<Number>BN\d{3}))'
czech_df_all = df['text'].str.extract(czech)
czech_df_all['Country'].unique()

#czech_df = czech_df_all[czech_df_all['Czech'] == 'Séc']
czech_df = czech_df_all[czech_df_all['Country'].notnull()]
czech_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>Country</th>
      <th>Number</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>93</th>
      <td>Séc--BN094</td>
      <td>Séc</td>
      <td>BN094</td>
    </tr>
    <tr>
      <th>114</th>
      <td>Séc--BN115</td>
      <td>Séc</td>
      <td>BN115</td>
    </tr>
    <tr>
      <th>203</th>
      <td>Séc--BN204</td>
      <td>Séc</td>
      <td>BN204</td>
    </tr>
  </tbody>
</table>
</div>




```python
turkey = r'((?P<Country>Thổ Nhĩ Kỳ)--(?P<Number>BN\d{3}))'
turkey_df_all = df['text'].str.extract(turkey)
turkey_df_all['Country'].unique()

turkey_df = turkey_df_all[turkey_df_all['Country'].notnull()]
turkey_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>Country</th>
      <th>Number</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>82</th>
      <td>Thổ Nhĩ Kỳ--BN083</td>
      <td>Thổ Nhĩ Kỳ</td>
      <td>BN083</td>
    </tr>
    <tr>
      <th>202</th>
      <td>Thổ Nhĩ Kỳ--BN203</td>
      <td>Thổ Nhĩ Kỳ</td>
      <td>BN203</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Concatenate all F0 cases
f0_total = pd.concat([f0_dropped, france_df, germany_df, netherlands_df, czech_df, turkey_df])
f0_total_final = f0_total.drop(axis=1, labels=[0])[['Number', 'Country']]
f0_total_final
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Number</th>
      <th>Country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>BN001</td>
      <td>Trung Quốc</td>
    </tr>
    <tr>
      <th>3</th>
      <td>BN004</td>
      <td>Trung Quốc</td>
    </tr>
    <tr>
      <th>4</th>
      <td>BN005</td>
      <td>Trung Quốc</td>
    </tr>
    <tr>
      <th>5</th>
      <td>BN006</td>
      <td>Trung Quốc</td>
    </tr>
    <tr>
      <th>6</th>
      <td>BN007</td>
      <td>Trung Quốc</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>93</th>
      <td>BN094</td>
      <td>Séc</td>
    </tr>
    <tr>
      <th>114</th>
      <td>BN115</td>
      <td>Séc</td>
    </tr>
    <tr>
      <th>203</th>
      <td>BN204</td>
      <td>Séc</td>
    </tr>
    <tr>
      <th>82</th>
      <td>BN083</td>
      <td>Thổ Nhĩ Kỳ</td>
    </tr>
    <tr>
      <th>202</th>
      <td>BN203</td>
      <td>Thổ Nhĩ Kỳ</td>
    </tr>
  </tbody>
</table>
<p>327 rows × 2 columns</p>
</div>




```python
f0_total_final['Country'].unique()
```




    array(['Trung Quốc', 'Anh', 'Hàn Quốc', 'Mỹ', 'Qatar', 'TBN', 'Malaysia',
           'Thụy Sĩ', 'Singapore', 'Nga', 'Hungary', 'Campuchia', 'Thái Lan',
           'Đan Mạch', 'Úc', 'Ả Rập', 'Nhật Bản', 'Canada', 'UAE',
           'Philippines', 'Mexico', 'Kuwait', 'Thụy Điển', 'Cameroon',
           'Bangladesh', 'Oman', 'Guinea-Bissau', 'Indonesia', 'Pháp',
           'Đức', 'Hà Lan', 'Séc', 'Thổ Nhĩ Kỳ'], dtype=object)




```python
f0_total_final.to_excel('import_f0.xlsx', index = False)
```

### F1


```python
#f1_rule = r'(?P<F1>-BN\d{3}-BN\d{3})'
f1_rule = r'(-(?P<Src>BN\d{3})-(?P<Des>BN\d{3}))'
f1 = df['text'].str.extract(f1_rule)
f1_valid = f1[f1[0].notnull()]
f1_valid
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>Src</th>
      <th>Des</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>-BN001-BN002</td>
      <td>BN001</td>
      <td>BN002</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-BN001-BN003</td>
      <td>BN001</td>
      <td>BN003</td>
    </tr>
    <tr>
      <th>9</th>
      <td>-BN006-BN010</td>
      <td>BN006</td>
      <td>BN010</td>
    </tr>
    <tr>
      <th>10</th>
      <td>-BN006-BN011</td>
      <td>BN006</td>
      <td>BN011</td>
    </tr>
    <tr>
      <th>11</th>
      <td>-BN006-BN012</td>
      <td>BN006</td>
      <td>BN012</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>664</th>
      <td>-BN448-BN665</td>
      <td>BN448</td>
      <td>BN665</td>
    </tr>
    <tr>
      <th>666</th>
      <td>-BN628-BN667</td>
      <td>BN628</td>
      <td>BN667</td>
    </tr>
    <tr>
      <th>667</th>
      <td>-BN628-BN668</td>
      <td>BN628</td>
      <td>BN668</td>
    </tr>
    <tr>
      <th>668</th>
      <td>-BN510-BN669</td>
      <td>BN510</td>
      <td>BN669</td>
    </tr>
    <tr>
      <th>670</th>
      <td>-BN469-BN671</td>
      <td>BN469</td>
      <td>BN671</td>
    </tr>
  </tbody>
</table>
<p>84 rows × 3 columns</p>
</div>




```python
f1_final = f1_valid.drop(axis=1, labels=[0])
f1_final
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Src</th>
      <th>Des</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>BN001</td>
      <td>BN002</td>
    </tr>
    <tr>
      <th>2</th>
      <td>BN001</td>
      <td>BN003</td>
    </tr>
    <tr>
      <th>9</th>
      <td>BN006</td>
      <td>BN010</td>
    </tr>
    <tr>
      <th>10</th>
      <td>BN006</td>
      <td>BN011</td>
    </tr>
    <tr>
      <th>11</th>
      <td>BN006</td>
      <td>BN012</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>664</th>
      <td>BN448</td>
      <td>BN665</td>
    </tr>
    <tr>
      <th>666</th>
      <td>BN628</td>
      <td>BN667</td>
    </tr>
    <tr>
      <th>667</th>
      <td>BN628</td>
      <td>BN668</td>
    </tr>
    <tr>
      <th>668</th>
      <td>BN510</td>
      <td>BN669</td>
    </tr>
    <tr>
      <th>670</th>
      <td>BN469</td>
      <td>BN671</td>
    </tr>
  </tbody>
</table>
<p>84 rows × 2 columns</p>
</div>




```python
f1_final.to_excel('import_f1f2.xlsx', index = False)
```

### Community


```python
community_rule = r'(^--(?P<Community>BN\d{3})$)'
community = df['text'].str.extract(community_rule)
community_valid = community[community['Community'].notnull()]
community_valid
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>Community</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>85</th>
      <td>--BN086</td>
      <td>BN086</td>
    </tr>
    <tr>
      <th>90</th>
      <td>--BN091</td>
      <td>BN091</td>
    </tr>
    <tr>
      <th>96</th>
      <td>--BN097</td>
      <td>BN097</td>
    </tr>
    <tr>
      <th>97</th>
      <td>--BN098</td>
      <td>BN098</td>
    </tr>
    <tr>
      <th>118</th>
      <td>--BN119</td>
      <td>BN119</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>659</th>
      <td>--BN660</td>
      <td>BN660</td>
    </tr>
    <tr>
      <th>660</th>
      <td>--BN661</td>
      <td>BN661</td>
    </tr>
    <tr>
      <th>661</th>
      <td>--BN662</td>
      <td>BN662</td>
    </tr>
    <tr>
      <th>665</th>
      <td>--BN666</td>
      <td>BN666</td>
    </tr>
    <tr>
      <th>669</th>
      <td>--BN670</td>
      <td>BN670</td>
    </tr>
  </tbody>
</table>
<p>260 rows × 2 columns</p>
</div>




```python
community_dropped = community_valid.drop(axis=1, labels=[0])
community_dropped
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Community</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>85</th>
      <td>BN086</td>
    </tr>
    <tr>
      <th>90</th>
      <td>BN091</td>
    </tr>
    <tr>
      <th>96</th>
      <td>BN097</td>
    </tr>
    <tr>
      <th>97</th>
      <td>BN098</td>
    </tr>
    <tr>
      <th>118</th>
      <td>BN119</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>659</th>
      <td>BN660</td>
    </tr>
    <tr>
      <th>660</th>
      <td>BN661</td>
    </tr>
    <tr>
      <th>661</th>
      <td>BN662</td>
    </tr>
    <tr>
      <th>665</th>
      <td>BN666</td>
    </tr>
    <tr>
      <th>669</th>
      <td>BN670</td>
    </tr>
  </tbody>
</table>
<p>260 rows × 1 columns</p>
</div>




```python
community_dropped.to_excel('community_00.xlsx', index = False)
```


```python
com_data = '["--BN001","--BN002","--BN003","--BN004","--BN005","Sơn Lôi, Vĩnh Phúc--BN006","--BN007","--BN008","--BN009","Sơn Lôi, Vĩnh Phúc--BN010","Sơn Lôi, Vĩnh Phúc--BN011","Sơn Lôi, Vĩnh Phúc--BN012","Sơn Lôi, Vĩnh Phúc--BN013","--BN014","Sơn Lôi, Vĩnh Phúc--BN015","Sơn Lôi, Vĩnh Phúc--BN016","Trúc Bạch, Hà Nội--BN017","--BN018","Trúc Bạch, Hà Nội--BN019","Trúc Bạch, Hà Nội--BN020","--BN021","--BN022","--BN023","--BN024","--BN025","--BN026","--BN027","--BN028","--BN029","--BN030","--BN031","--BN032","--BN033","Phan Thiết, Bình Thuận--BN034","--BN035","Phan Thiết, Bình Thuận--BN036","Phan Thiết, Bình Thuận--BN037","Phan Thiết, Bình Thuận--BN038","Phan Thiết, Bình Thuận--BN039","Phan Thiết, Bình Thuận--BN040","Phan Thiết, Bình Thuận--BN041","Phan Thiết, Bình Thuận--BN042","Phan Thiết, Bình Thuận--BN043","--BN044","Phan Thiết, Bình Thuận--BN045","--BN046","Trúc Bạch, Hà Nội--BN047","Phan Thiết, Bình Thuận--BN048","--BN049","--BN050","--BN051","--BN052","--BN053","--BN054","--BN055","--BN056","--BN057","--BN058","--BN059","--BN060","--BN061","--BN062","--BN063","--BN064","Phan Thiết, Bình Thuận--BN065","--BN066","--BN067","--BN068","--BN069","--BN070","--BN071","--BN072","--BN073","--BN074","--BN075","--BN076","--BN077","--BN078","--BN079","--BN080","--BN081","--BN082","--BN083","--BN084","--BN085","BV Bạch Mai--BN086","BV Bạch Mai--BN087","--BN088","--BN089","--BN090","Buddha Bar & Grill--BN091","--BN092","--BN093","--BN094","--BN095","--BN096","Buddha Bar & Grill--BN097","Buddha Bar & Grill--BN098","--BN099","--BN100","--BN101","--BN102","--BN103","--BN104","--BN105","--BN106","BV Bạch Mai--BN107","--BN108","--BN109","--BN110","--BN111","--BN112","--BN113","--BN114","--BN115","--BN116","--BN117","--BN118","--BN119","Buddha Bar & Grill--BN120","--BN121","--BN122","--BN123","Buddha Bar & Grill--BN124","Buddha Bar & Grill--BN125","Buddha Bar & Grill--BN126","Buddha Bar & Grill--BN127","--BN128","--BN129","--BN130","--BN131","--BN132","BV Bạch Mai--BN133","--BN134","--BN135","--BN136","--BN137","--BN138","--BN139","--BN140","--BN141","--BN142","--BN143","--BN144","--BN145","--BN146","--BN147","--BN148","--BN149","--BN150","Buddha Bar & Grill--BN151","Buddha Bar & Grill--BN152","--BN153","--BN154","--BN155","--BN156","Buddha Bar & Grill--BN157","Buddha Bar & Grill--BN158","Buddha Bar & Grill--BN159","--BN160","BV Bạch Mai--BN161","BV Bạch Mai--BN162","BV Bạch Mai--BN163","--BN164","--BN165","--BN166","--BN167","Trường Sinh--BN168","Trường Sinh--BN169","BV Bạch Mai--BN170","--BN171","BV Bạch Mai--BN172","--BN173","Trường Sinh--BN174","Trường Sinh--BN175","Trường Sinh--BN176","Trường Sinh--BN177","Trường Sinh--BN178","--BN179","--BN180","--BN181","--BN182","--BN183","Trường Sinh--BN184","BV Bạch Mai--BN185","--BN186","--BN187","Trường Sinh--BN188","Trường Sinh--BN189","Trường Sinh--BN190","Trường Sinh--BN191","Trường Sinh--BN192","Trường Sinh--BN193","Trường Sinh--BN194","Trường Sinh--BN195","Trường Sinh--BN196","BV Bạch Mai--BN197","Trường Sinh--BN198","Trường Sinh--BN199","Trường Sinh--BN200","Trường Sinh--BN201","Trường Sinh--BN202","--BN203","--BN204","Trường Sinh--BN205","Buddha Bar & Grill--BN206","Buddha Bar & Grill--BN207","Trường Sinh--BN208","BV Bạch Mai--BN209","--BN210","--BN211","--BN212","BV Bạch Mai--BN213","Trường Sinh--BN214","Trường Sinh--BN215","--BN216","--BN217","--BN218","BV Bạch Mai--BN219","--BN220","--BN221","--BN222","BV Bạch Mai--BN223","Buddha Bar & Grill--BN224","--BN225","--BN226","Trường Sinh--BN227","--BN228","--BN229","--BN230","Trường Sinh--BN231","--BN232","--BN233","--BN234","Buddha Bar & Grill--BN235","Buddha Bar & Grill--BN236","--BN237","--BN238","BV Bạch Mai--BN239","--BN240","--BN241","--BN242","Hạ Lôi, Mê Linh--BN243","--BN244","--BN245","--BN246","Buddha Bar & Grill--BN247","--BN248","--BN249","Hạ Lôi, Mê Linh--BN250","--BN251","--BN252","Hạ Lôi, Mê Linh--BN253","Hạ Lôi, Mê Linh--BN254","--BN255","--BN256","Hạ Lôi, Mê Linh--BN257","Hạ Lôi, Mê Linh--BN258","Hạ Lôi, Mê Linh--BN259","Hạ Lôi, Mê Linh--BN260","Hạ Lôi, Mê Linh--BN261","Hạ Lôi, Mê Linh--BN262","Hạ Lôi, Mê Linh--BN263","Hạ Lôi, Mê Linh--BN264","--BN265","BV Bạch Mai--BN266","Hạ Lôi, Mê Linh--BN267","--BN268","--BN269","--BN270","--BN271","--BN272","--BN273","--BN274","--BN275","--BN276","--BN277","--BN278","--BN279","--BN280","--BN281","--BN282","--BN283","--BN284","--BN285","--BN286","--BN287","--BN288","--BN289","--BN290","--BN291","--BN292","--BN293","--BN294","--BN295","--BN296","--BN297","--BN298","--BN299","--BN300","--BN301","--BN302","--BN303","--BN304","--BN305","--BN306","--BN307","--BN308","--BN309","--BN310","--BN311","--BN312","--BN313","--BN314","--BN315","--BN316","--BN317","--BN318","--BN319","--BN320","--BN321","--BN322","--BN323","--BN324","--BN325","--BN326","--BN327","--BN328","--BN329","--BN330","--BN331","--BN332","--BN333","--BN334","--BN335","--BN336","--BN337","--BN338","--BN339","--BN340","--BN341","--BN342","--BN343","--BN344","--BN345","--BN346","--BN347","--BN348","--BN349","--BN350","--BN351","--BN352","--BN353","--BN354","--BN355","--BN356","--BN357","--BN358","--BN359","--BN360","--BN361","--BN362","--BN363","--BN364","--BN365","--BN366","--BN367","--BN368","--BN369","--BN370","--BN371","--BN372","--BN373","--BN374","--BN375","--BN376","--BN377","--BN378","--BN379","--BN380","--BN381","--BN382","--BN383","--BN384","--BN385","--BN386","--BN387","--BN388","--BN389","--BN390","--BN391","--BN392","--BN393","--BN394","--BN395","--BN396","--BN397","--BN398","--BN399","--BN400","--BN401","--BN402","--BN403","--BN404","--BN405","--BN406","--BN407","--BN408","--BN409","--BN410","--BN411","--BN412","--BN413","--BN414","--BN415","BV Đà Nẵng--BN416","--BN417","BV Đà Nẵng--BN418","BV Đà Nẵng--BN419","BV Đà Nẵng--BN420","BV Đà Nẵng--BN421","BV Đà Nẵng--BN422","BV Đà Nẵng--BN423","BV Đà Nẵng--BN424","BV Đà Nẵng--BN425","BV Đà Nẵng--BN426","BV Đà Nẵng--BN427","BV Đà Nẵng--BN428","BV Đà Nẵng--BN429","BV Đà Nẵng--BN430","BV Đà Nẵng--BN431","BV Đà Nẵng--BN432","BV Đà Nẵng--BN433","BV Đà Nẵng--BN434","BV Đà Nẵng--BN435","BV Đà Nẵng--BN436","BV Đà Nẵng--BN437","BV Đà Nẵng--BN438","BV Đà Nẵng--BN439","BV Đà Nẵng--BN440","BV Đà Nẵng--BN441","BV Đà Nẵng--BN442","BV Đà Nẵng--BN443","BV Đà Nẵng--BN444","BV Đà Nẵng--BN445","BV Đà Nẵng--BN446","Chưa xác định--BN447","BV Đà Nẵng--BN448","BV Đà Nẵng--BN449","BV Đà Nẵng--BN450","BV Đà Nẵng--BN451","BV Đà Nẵng--BN452","BV Đà Nẵng--BN453","BV Đà Nẵng--BN454","BV Đà Nẵng--BN455","BV Đà Nẵng--BN456","BV Đà Nẵng--BN457","BV Đà Nẵng--BN458","BV C Đà Nẵng--BN459","BV Đà Nẵng--BN460","BV Đà Nẵng--BN461","BV Đà Nẵng--BN462","BV Đà Nẵng--BN463","BV Đà Nẵng--BN464","BV Đà Nẵng--BN465","BV Đà Nẵng--BN466","BV Đà Nẵng--BN467","BV Đà Nẵng--BN468","BV Đà Nẵng--BN469","BV Đà Nẵng--BN470","BV Đà Nẵng--BN471","BV Đà Nẵng--BN472","BV Đà Nẵng--BN473","BV Đà Nẵng--BN474","BV Đà Nẵng--BN475","BV Đà Nẵng--BN476","BV Đà Nẵng--BN477","BV Đà Nẵng--BN478","BV Đà Nẵng--BN479","BV Đà Nẵng--BN480","BV Đà Nẵng--BN481","BV Đà Nẵng--BN482","BV Đà Nẵng--BN483","BV Đà Nẵng--BN484","BV Đà Nẵng--BN485","BV Đà Nẵng--BN486","BV Đà Nẵng--BN487","BV Đà Nẵng--BN488","BV Đà Nẵng--BN489","BV Đà Nẵng--BN490","BV Đà Nẵng--BN491","BV Đà Nẵng--BN492","BV Đà Nẵng--BN493","BV Đà Nẵng--BN494","BV Đà Nẵng--BN495","BV Đà Nẵng--BN496","BV Đà Nẵng--BN497","BV Đà Nẵng--BN498","BV Đà Nẵng--BN499","BV phổi Đà Nẵng--BN500","BV phổi Đà Nẵng--BN501","BV Đà Nẵng--BN502","BV Đà Nẵng--BN503","Chưa xác định--BN504","BV Đà Nẵng--BN505","BV Đà Nẵng--BN506","BV Đà Nẵng--BN507","BV Đà Nẵng--BN508","Chưa xác định--BN509","BV Đà Nẵng--BN510","--BN511","--BN512","--BN513","--BN514","--BN515","--BN516","BV Đà Nẵng--BN517","BV Đà Nẵng--BN518","BV Đà Nẵng--BN519","BV Đà Nẵng--BN520","Chưa xác định--BN521","BV Đà Nẵng--BN522","BV Đà Nẵng--BN523","BV Đà Nẵng--BN524","BV Đà Nẵng--BN525","BV Đà Nẵng--BN526","--BN527","--BN528","--BN529","--BN530","--BN531","--BN532","--BN533","--BN534","--BN535","--BN536","--BN537","--BN538","--BN539","--BN540","--BN541","--BN542","--BN543","--BN544","--BN545","--BN546","BV Đà Nẵng--BN547","BV Đà Nẵng--BN548","BV Đà Nẵng--BN549","BV Đà Nẵng--BN550","BV Đà Nẵng--BN551","BV Đà Nẵng--BN552","BV Đà Nẵng--BN553","BV Đà Nẵng--BN554","BV Đà Nẵng--BN555","BV Đà Nẵng--BN556","BV Đà Nẵng--BN557","BV Đà Nẵng--BN558","--BN559","--BN560","BV Đà Nẵng--BN561","BV Đà Nẵng--BN562","BV Đà Nẵng--BN563","BV Đà Nẵng--BN564","BV Đà Nẵng--BN565","BV Đà Nẵng--BN566","BV Đà Nẵng--BN567","BV Đà Nẵng--BN568","BV Đà Nẵng--BN569","BV Đà Nẵng--BN570","BV Đà Nẵng--BN571","BV Đà Nẵng--BN572","BV Đà Nẵng--BN573","BV Đà Nẵng--BN574","BV Đà Nẵng--BN575","BV Đà Nẵng--BN576","BV Đà Nẵng--BN577","BV Đà Nẵng--BN578","BV Đà Nẵng--BN579","BV Đà Nẵng--BN580","BV Đà Nẵng--BN581","BV Đà Nẵng--BN582","BV Đà Nẵng--BN583","BV Đà Nẵng--BN584","BV Đà Nẵng--BN585","BV Đà Nẵng--BN586","--BN587","--BN588","Đà Nẵng--BN589","BV Đà Nẵng--BN590","--BN591","BV Đà Nẵng--BN592","BV Đà Nẵng--BN593","BV Đà Nẵng--BN594","BV Đà Nẵng--BN595","BV C Đà Nẵng--BN596","BV Đà Nẵng--BN597","Khu cư trú--BN598","Khu cư trú--BN599","Khu cư trú--BN600","Tiệc cưới For You--BN601","Tiệc cưới For You--BN602","--BN603","--BN604","--BN605","--BN606","--BN607","BV Đà Nẵng--BN608","--BN609","--BN610","--BN611","BV Đà Nẵng--BN612","--BN613","--BN614","--BN615","--BN616","--BN617","--BN618","--BN619","--BN620","--BN621","BV Đà Nẵng--BN622","--BN623","BV Đà Nẵng--BN624","BV Đà Nẵng--BN625","--BN626","--BN627","BV Đà Nẵng--BN628","Khu cư trú--BN629","Khu cư trú--BN630","BV Đà Nẵng--BN631","BV Đà Nẵng--BN632","BV Đà Nẵng--BN633","BV Đà Nẵng--BN634","BV C Đà Nẵng--BN635","Đám tang HCG--BN636","Đám tang HCG--BN637","Đám tang HCG--BN638","BV Đà Nẵng--BN639","Đám tang HCG--BN640","BV Đà Nẵng--BN641","BV Đà Nẵng--BN642","BV Đà Nẵng--BN643","BV Đà Nẵng--BN644","BV Đà Nẵng--BN645","BV Đà Nẵng--BN646","--BN647","--BN648","--BN649","--BN650","--BN651","--BN652","BV Phụ sản- Nhi Đà Nẵng--BN653","Bến xe TT Đà Nẵng--BN654","--BN655","--BN656","--BN657","BV Đà Nẵng--BN658","BV Đà Nẵng--BN659","BV Đà Nẵng--BN660","BV Đà Nẵng--BN661","BV Đà Nẵng--BN662","--BN663","--BN664","--BN665","BV Đà Nẵng--BN666","--BN667","--BN668","BV Đà Nẵng--BN669","--BN670","--BN671"]'
```


```python
com_data
```




    '["--BN001","--BN002","--BN003","--BN004","--BN005","Sơn Lôi, Vĩnh Phúc--BN006","--BN007","--BN008","--BN009","Sơn Lôi, Vĩnh Phúc--BN010","Sơn Lôi, Vĩnh Phúc--BN011","Sơn Lôi, Vĩnh Phúc--BN012","Sơn Lôi, Vĩnh Phúc--BN013","--BN014","Sơn Lôi, Vĩnh Phúc--BN015","Sơn Lôi, Vĩnh Phúc--BN016","Trúc Bạch, Hà Nội--BN017","--BN018","Trúc Bạch, Hà Nội--BN019","Trúc Bạch, Hà Nội--BN020","--BN021","--BN022","--BN023","--BN024","--BN025","--BN026","--BN027","--BN028","--BN029","--BN030","--BN031","--BN032","--BN033","Phan Thiết, Bình Thuận--BN034","--BN035","Phan Thiết, Bình Thuận--BN036","Phan Thiết, Bình Thuận--BN037","Phan Thiết, Bình Thuận--BN038","Phan Thiết, Bình Thuận--BN039","Phan Thiết, Bình Thuận--BN040","Phan Thiết, Bình Thuận--BN041","Phan Thiết, Bình Thuận--BN042","Phan Thiết, Bình Thuận--BN043","--BN044","Phan Thiết, Bình Thuận--BN045","--BN046","Trúc Bạch, Hà Nội--BN047","Phan Thiết, Bình Thuận--BN048","--BN049","--BN050","--BN051","--BN052","--BN053","--BN054","--BN055","--BN056","--BN057","--BN058","--BN059","--BN060","--BN061","--BN062","--BN063","--BN064","Phan Thiết, Bình Thuận--BN065","--BN066","--BN067","--BN068","--BN069","--BN070","--BN071","--BN072","--BN073","--BN074","--BN075","--BN076","--BN077","--BN078","--BN079","--BN080","--BN081","--BN082","--BN083","--BN084","--BN085","BV Bạch Mai--BN086","BV Bạch Mai--BN087","--BN088","--BN089","--BN090","Buddha Bar & Grill--BN091","--BN092","--BN093","--BN094","--BN095","--BN096","Buddha Bar & Grill--BN097","Buddha Bar & Grill--BN098","--BN099","--BN100","--BN101","--BN102","--BN103","--BN104","--BN105","--BN106","BV Bạch Mai--BN107","--BN108","--BN109","--BN110","--BN111","--BN112","--BN113","--BN114","--BN115","--BN116","--BN117","--BN118","--BN119","Buddha Bar & Grill--BN120","--BN121","--BN122","--BN123","Buddha Bar & Grill--BN124","Buddha Bar & Grill--BN125","Buddha Bar & Grill--BN126","Buddha Bar & Grill--BN127","--BN128","--BN129","--BN130","--BN131","--BN132","BV Bạch Mai--BN133","--BN134","--BN135","--BN136","--BN137","--BN138","--BN139","--BN140","--BN141","--BN142","--BN143","--BN144","--BN145","--BN146","--BN147","--BN148","--BN149","--BN150","Buddha Bar & Grill--BN151","Buddha Bar & Grill--BN152","--BN153","--BN154","--BN155","--BN156","Buddha Bar & Grill--BN157","Buddha Bar & Grill--BN158","Buddha Bar & Grill--BN159","--BN160","BV Bạch Mai--BN161","BV Bạch Mai--BN162","BV Bạch Mai--BN163","--BN164","--BN165","--BN166","--BN167","Trường Sinh--BN168","Trường Sinh--BN169","BV Bạch Mai--BN170","--BN171","BV Bạch Mai--BN172","--BN173","Trường Sinh--BN174","Trường Sinh--BN175","Trường Sinh--BN176","Trường Sinh--BN177","Trường Sinh--BN178","--BN179","--BN180","--BN181","--BN182","--BN183","Trường Sinh--BN184","BV Bạch Mai--BN185","--BN186","--BN187","Trường Sinh--BN188","Trường Sinh--BN189","Trường Sinh--BN190","Trường Sinh--BN191","Trường Sinh--BN192","Trường Sinh--BN193","Trường Sinh--BN194","Trường Sinh--BN195","Trường Sinh--BN196","BV Bạch Mai--BN197","Trường Sinh--BN198","Trường Sinh--BN199","Trường Sinh--BN200","Trường Sinh--BN201","Trường Sinh--BN202","--BN203","--BN204","Trường Sinh--BN205","Buddha Bar & Grill--BN206","Buddha Bar & Grill--BN207","Trường Sinh--BN208","BV Bạch Mai--BN209","--BN210","--BN211","--BN212","BV Bạch Mai--BN213","Trường Sinh--BN214","Trường Sinh--BN215","--BN216","--BN217","--BN218","BV Bạch Mai--BN219","--BN220","--BN221","--BN222","BV Bạch Mai--BN223","Buddha Bar & Grill--BN224","--BN225","--BN226","Trường Sinh--BN227","--BN228","--BN229","--BN230","Trường Sinh--BN231","--BN232","--BN233","--BN234","Buddha Bar & Grill--BN235","Buddha Bar & Grill--BN236","--BN237","--BN238","BV Bạch Mai--BN239","--BN240","--BN241","--BN242","Hạ Lôi, Mê Linh--BN243","--BN244","--BN245","--BN246","Buddha Bar & Grill--BN247","--BN248","--BN249","Hạ Lôi, Mê Linh--BN250","--BN251","--BN252","Hạ Lôi, Mê Linh--BN253","Hạ Lôi, Mê Linh--BN254","--BN255","--BN256","Hạ Lôi, Mê Linh--BN257","Hạ Lôi, Mê Linh--BN258","Hạ Lôi, Mê Linh--BN259","Hạ Lôi, Mê Linh--BN260","Hạ Lôi, Mê Linh--BN261","Hạ Lôi, Mê Linh--BN262","Hạ Lôi, Mê Linh--BN263","Hạ Lôi, Mê Linh--BN264","--BN265","BV Bạch Mai--BN266","Hạ Lôi, Mê Linh--BN267","--BN268","--BN269","--BN270","--BN271","--BN272","--BN273","--BN274","--BN275","--BN276","--BN277","--BN278","--BN279","--BN280","--BN281","--BN282","--BN283","--BN284","--BN285","--BN286","--BN287","--BN288","--BN289","--BN290","--BN291","--BN292","--BN293","--BN294","--BN295","--BN296","--BN297","--BN298","--BN299","--BN300","--BN301","--BN302","--BN303","--BN304","--BN305","--BN306","--BN307","--BN308","--BN309","--BN310","--BN311","--BN312","--BN313","--BN314","--BN315","--BN316","--BN317","--BN318","--BN319","--BN320","--BN321","--BN322","--BN323","--BN324","--BN325","--BN326","--BN327","--BN328","--BN329","--BN330","--BN331","--BN332","--BN333","--BN334","--BN335","--BN336","--BN337","--BN338","--BN339","--BN340","--BN341","--BN342","--BN343","--BN344","--BN345","--BN346","--BN347","--BN348","--BN349","--BN350","--BN351","--BN352","--BN353","--BN354","--BN355","--BN356","--BN357","--BN358","--BN359","--BN360","--BN361","--BN362","--BN363","--BN364","--BN365","--BN366","--BN367","--BN368","--BN369","--BN370","--BN371","--BN372","--BN373","--BN374","--BN375","--BN376","--BN377","--BN378","--BN379","--BN380","--BN381","--BN382","--BN383","--BN384","--BN385","--BN386","--BN387","--BN388","--BN389","--BN390","--BN391","--BN392","--BN393","--BN394","--BN395","--BN396","--BN397","--BN398","--BN399","--BN400","--BN401","--BN402","--BN403","--BN404","--BN405","--BN406","--BN407","--BN408","--BN409","--BN410","--BN411","--BN412","--BN413","--BN414","--BN415","BV Đà Nẵng--BN416","--BN417","BV Đà Nẵng--BN418","BV Đà Nẵng--BN419","BV Đà Nẵng--BN420","BV Đà Nẵng--BN421","BV Đà Nẵng--BN422","BV Đà Nẵng--BN423","BV Đà Nẵng--BN424","BV Đà Nẵng--BN425","BV Đà Nẵng--BN426","BV Đà Nẵng--BN427","BV Đà Nẵng--BN428","BV Đà Nẵng--BN429","BV Đà Nẵng--BN430","BV Đà Nẵng--BN431","BV Đà Nẵng--BN432","BV Đà Nẵng--BN433","BV Đà Nẵng--BN434","BV Đà Nẵng--BN435","BV Đà Nẵng--BN436","BV Đà Nẵng--BN437","BV Đà Nẵng--BN438","BV Đà Nẵng--BN439","BV Đà Nẵng--BN440","BV Đà Nẵng--BN441","BV Đà Nẵng--BN442","BV Đà Nẵng--BN443","BV Đà Nẵng--BN444","BV Đà Nẵng--BN445","BV Đà Nẵng--BN446","Chưa xác định--BN447","BV Đà Nẵng--BN448","BV Đà Nẵng--BN449","BV Đà Nẵng--BN450","BV Đà Nẵng--BN451","BV Đà Nẵng--BN452","BV Đà Nẵng--BN453","BV Đà Nẵng--BN454","BV Đà Nẵng--BN455","BV Đà Nẵng--BN456","BV Đà Nẵng--BN457","BV Đà Nẵng--BN458","BV C Đà Nẵng--BN459","BV Đà Nẵng--BN460","BV Đà Nẵng--BN461","BV Đà Nẵng--BN462","BV Đà Nẵng--BN463","BV Đà Nẵng--BN464","BV Đà Nẵng--BN465","BV Đà Nẵng--BN466","BV Đà Nẵng--BN467","BV Đà Nẵng--BN468","BV Đà Nẵng--BN469","BV Đà Nẵng--BN470","BV Đà Nẵng--BN471","BV Đà Nẵng--BN472","BV Đà Nẵng--BN473","BV Đà Nẵng--BN474","BV Đà Nẵng--BN475","BV Đà Nẵng--BN476","BV Đà Nẵng--BN477","BV Đà Nẵng--BN478","BV Đà Nẵng--BN479","BV Đà Nẵng--BN480","BV Đà Nẵng--BN481","BV Đà Nẵng--BN482","BV Đà Nẵng--BN483","BV Đà Nẵng--BN484","BV Đà Nẵng--BN485","BV Đà Nẵng--BN486","BV Đà Nẵng--BN487","BV Đà Nẵng--BN488","BV Đà Nẵng--BN489","BV Đà Nẵng--BN490","BV Đà Nẵng--BN491","BV Đà Nẵng--BN492","BV Đà Nẵng--BN493","BV Đà Nẵng--BN494","BV Đà Nẵng--BN495","BV Đà Nẵng--BN496","BV Đà Nẵng--BN497","BV Đà Nẵng--BN498","BV Đà Nẵng--BN499","BV phổi Đà Nẵng--BN500","BV phổi Đà Nẵng--BN501","BV Đà Nẵng--BN502","BV Đà Nẵng--BN503","Chưa xác định--BN504","BV Đà Nẵng--BN505","BV Đà Nẵng--BN506","BV Đà Nẵng--BN507","BV Đà Nẵng--BN508","Chưa xác định--BN509","BV Đà Nẵng--BN510","--BN511","--BN512","--BN513","--BN514","--BN515","--BN516","BV Đà Nẵng--BN517","BV Đà Nẵng--BN518","BV Đà Nẵng--BN519","BV Đà Nẵng--BN520","Chưa xác định--BN521","BV Đà Nẵng--BN522","BV Đà Nẵng--BN523","BV Đà Nẵng--BN524","BV Đà Nẵng--BN525","BV Đà Nẵng--BN526","--BN527","--BN528","--BN529","--BN530","--BN531","--BN532","--BN533","--BN534","--BN535","--BN536","--BN537","--BN538","--BN539","--BN540","--BN541","--BN542","--BN543","--BN544","--BN545","--BN546","BV Đà Nẵng--BN547","BV Đà Nẵng--BN548","BV Đà Nẵng--BN549","BV Đà Nẵng--BN550","BV Đà Nẵng--BN551","BV Đà Nẵng--BN552","BV Đà Nẵng--BN553","BV Đà Nẵng--BN554","BV Đà Nẵng--BN555","BV Đà Nẵng--BN556","BV Đà Nẵng--BN557","BV Đà Nẵng--BN558","--BN559","--BN560","BV Đà Nẵng--BN561","BV Đà Nẵng--BN562","BV Đà Nẵng--BN563","BV Đà Nẵng--BN564","BV Đà Nẵng--BN565","BV Đà Nẵng--BN566","BV Đà Nẵng--BN567","BV Đà Nẵng--BN568","BV Đà Nẵng--BN569","BV Đà Nẵng--BN570","BV Đà Nẵng--BN571","BV Đà Nẵng--BN572","BV Đà Nẵng--BN573","BV Đà Nẵng--BN574","BV Đà Nẵng--BN575","BV Đà Nẵng--BN576","BV Đà Nẵng--BN577","BV Đà Nẵng--BN578","BV Đà Nẵng--BN579","BV Đà Nẵng--BN580","BV Đà Nẵng--BN581","BV Đà Nẵng--BN582","BV Đà Nẵng--BN583","BV Đà Nẵng--BN584","BV Đà Nẵng--BN585","BV Đà Nẵng--BN586","--BN587","--BN588","Đà Nẵng--BN589","BV Đà Nẵng--BN590","--BN591","BV Đà Nẵng--BN592","BV Đà Nẵng--BN593","BV Đà Nẵng--BN594","BV Đà Nẵng--BN595","BV C Đà Nẵng--BN596","BV Đà Nẵng--BN597","Khu cư trú--BN598","Khu cư trú--BN599","Khu cư trú--BN600","Tiệc cưới For You--BN601","Tiệc cưới For You--BN602","--BN603","--BN604","--BN605","--BN606","--BN607","BV Đà Nẵng--BN608","--BN609","--BN610","--BN611","BV Đà Nẵng--BN612","--BN613","--BN614","--BN615","--BN616","--BN617","--BN618","--BN619","--BN620","--BN621","BV Đà Nẵng--BN622","--BN623","BV Đà Nẵng--BN624","BV Đà Nẵng--BN625","--BN626","--BN627","BV Đà Nẵng--BN628","Khu cư trú--BN629","Khu cư trú--BN630","BV Đà Nẵng--BN631","BV Đà Nẵng--BN632","BV Đà Nẵng--BN633","BV Đà Nẵng--BN634","BV C Đà Nẵng--BN635","Đám tang HCG--BN636","Đám tang HCG--BN637","Đám tang HCG--BN638","BV Đà Nẵng--BN639","Đám tang HCG--BN640","BV Đà Nẵng--BN641","BV Đà Nẵng--BN642","BV Đà Nẵng--BN643","BV Đà Nẵng--BN644","BV Đà Nẵng--BN645","BV Đà Nẵng--BN646","--BN647","--BN648","--BN649","--BN650","--BN651","--BN652","BV Phụ sản- Nhi Đà Nẵng--BN653","Bến xe TT Đà Nẵng--BN654","--BN655","--BN656","--BN657","BV Đà Nẵng--BN658","BV Đà Nẵng--BN659","BV Đà Nẵng--BN660","BV Đà Nẵng--BN661","BV Đà Nẵng--BN662","--BN663","--BN664","--BN665","BV Đà Nẵng--BN666","--BN667","--BN668","BV Đà Nẵng--BN669","--BN670","--BN671"]'




```python
cells2 = re.findall(r'"(.*?)"', com_data)
df2 = pd.DataFrame(cells2, columns=['text'])
df2
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>--BN001</td>
    </tr>
    <tr>
      <th>1</th>
      <td>--BN002</td>
    </tr>
    <tr>
      <th>2</th>
      <td>--BN003</td>
    </tr>
    <tr>
      <th>3</th>
      <td>--BN004</td>
    </tr>
    <tr>
      <th>4</th>
      <td>--BN005</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>666</th>
      <td>--BN667</td>
    </tr>
    <tr>
      <th>667</th>
      <td>--BN668</td>
    </tr>
    <tr>
      <th>668</th>
      <td>BV Đà Nẵng--BN669</td>
    </tr>
    <tr>
      <th>669</th>
      <td>--BN670</td>
    </tr>
    <tr>
      <th>670</th>
      <td>--BN671</td>
    </tr>
  </tbody>
</table>
<p>671 rows × 1 columns</p>
</div>




```python
places = ['Sơn Lôi, Vĩnh Phúc', 'Hạ Lôi, Mê Linh', 'Phan Thiết, Bình Thuận', 'BV Đà Nẵng', 'Bến xe TT Đà Nẵng', 'Đám tang HCG', 'Trúc Bạch, Hà Nội',
         'BV Bạch Mai', 'BV C Đà Nẵng', 'BV phổi Đà Nẵng', 'BV Phụ sản- Nhi Đà Nẵng', 'Tiệc cưới For You', 'Trường Sinh', 'Khu cư trú', 'Đà Nẵng', 
         'Buddha Bar & Grill', 'Chưa xác định']
```


```python
rules = []
address = r'BN\d{3}'
for place in places:
    rule = '(?P<Place>' + place + ')--(?P<Number>' + address + ')'
    rules.append(rule)
rules
```




    ['(?P<Place>Sơn Lôi, Vĩnh Phúc)--(?P<Number>BN\\d{3})',
     '(?P<Place>Hạ Lôi, Mê Linh)--(?P<Number>BN\\d{3})',
     '(?P<Place>Phan Thiết, Bình Thuận)--(?P<Number>BN\\d{3})',
     '(?P<Place>BV Đà Nẵng)--(?P<Number>BN\\d{3})',
     '(?P<Place>Bến xe TT Đà Nẵng)--(?P<Number>BN\\d{3})',
     '(?P<Place>Đám tang HCG)--(?P<Number>BN\\d{3})',
     '(?P<Place>Trúc Bạch, Hà Nội)--(?P<Number>BN\\d{3})',
     '(?P<Place>BV Bạch Mai)--(?P<Number>BN\\d{3})',
     '(?P<Place>BV C Đà Nẵng)--(?P<Number>BN\\d{3})',
     '(?P<Place>BV phổi Đà Nẵng)--(?P<Number>BN\\d{3})',
     '(?P<Place>BV Phụ sản- Nhi Đà Nẵng)--(?P<Number>BN\\d{3})',
     '(?P<Place>Tiệc cưới For You)--(?P<Number>BN\\d{3})',
     '(?P<Place>Trường Sinh)--(?P<Number>BN\\d{3})',
     '(?P<Place>Khu cư trú)--(?P<Number>BN\\d{3})',
     '(?P<Place>Đà Nẵng)--(?P<Number>BN\\d{3})',
     '(?P<Place>Buddha Bar & Grill)--(?P<Number>BN\\d{3})',
     '(?P<Place>Chưa xác định)--(?P<Number>BN\\d{3})']




```python
rules = [r'(?P<Place>Sơn Lôi, Vĩnh Phúc)--(?P<Number>BN\d{3})',
 r'(?P<Place>Hạ Lôi, Mê Linh)--(?P<Number>BN\d{3})',
 r'(?P<Place>Phan Thiết, Bình Thuận)--(?P<Number>BN\d{3})',
 r'(?P<Place>BV Đà Nẵng)--(?P<Number>BN\d{3})',
 r'(?P<Place>Bến xe TT Đà Nẵng)--(?P<Number>BN\d{3})',
 r'(?P<Place>Đám tang HCG)--(?P<Number>BN\d{3})',
 r'(?P<Place>Trúc Bạch, Hà Nội)--(?P<Number>BN\d{3})',
 r'(?P<Place>BV Bạch Mai)--(?P<Number>BN\d{3})',
 r'(?P<Place>BV C Đà Nẵng)--(?P<Number>BN\d{3})',
 r'(?P<Place>BV phổi Đà Nẵng)--(?P<Number>BN\d{3})',
 r'(?P<Place>BV Phụ sản- Nhi Đà Nẵng)--(?P<Number>BN\d{3})',
 r'(?P<Place>Tiệc cưới For You)--(?P<Number>BN\d{3})',
 r'(?P<Place>Trường Sinh)--(?P<Number>BN\d{3})',
 r'(?P<Place>Khu cư trú)--(?P<Number>BN\d{3})',
 r'(?P<Place>Đà Nẵng)--(?P<Number>BN\d{3})',
 r'(?P<Place>Buddha Bar & Grill)--(?P<Number>BN\d{3})',
 r'(?P<Place>Chưa xác định)--(?P<Number>BN\d{3})']
```


```python
from functools import reduce
ext = [df2['text'].str.extract(rule) for rule in rules]
ext_df = reduce(lambda x,y: x.fillna(y),ext)
ext_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Place</th>
      <th>Number</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>666</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>667</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>668</th>
      <td>BV Đà Nẵng</td>
      <td>BN669</td>
    </tr>
    <tr>
      <th>669</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>670</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>671 rows × 2 columns</p>
</div>




```python
ext_df_valid = extdf[extdf['Place'].notnull()]
ext_df_valid_final = ext_df_valid[['Number', 'Place']]
ext_df_valid_final
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Number</th>
      <th>Place</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>BN006</td>
      <td>Sơn Lôi, Vĩnh Phúc</td>
    </tr>
    <tr>
      <th>9</th>
      <td>BN010</td>
      <td>Sơn Lôi, Vĩnh Phúc</td>
    </tr>
    <tr>
      <th>10</th>
      <td>BN011</td>
      <td>Sơn Lôi, Vĩnh Phúc</td>
    </tr>
    <tr>
      <th>11</th>
      <td>BN012</td>
      <td>Sơn Lôi, Vĩnh Phúc</td>
    </tr>
    <tr>
      <th>12</th>
      <td>BN013</td>
      <td>Sơn Lôi, Vĩnh Phúc</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>659</th>
      <td>BN660</td>
      <td>BV Đà Nẵng</td>
    </tr>
    <tr>
      <th>660</th>
      <td>BN661</td>
      <td>BV Đà Nẵng</td>
    </tr>
    <tr>
      <th>661</th>
      <td>BN662</td>
      <td>BV Đà Nẵng</td>
    </tr>
    <tr>
      <th>665</th>
      <td>BN666</td>
      <td>BV Đà Nẵng</td>
    </tr>
    <tr>
      <th>668</th>
      <td>BN669</td>
      <td>BV Đà Nẵng</td>
    </tr>
  </tbody>
</table>
<p>288 rows × 2 columns</p>
</div>




```python
ext_df_valid_final.to_excel('domestic_identified.xlsx', index = False)
```


```python
unid_rule = r'^--(?P<Number>BN\d{3})$'
unid_df = df2['text'].str.extract(unid_rule)
unid_df_valid = unid_df[unid_df['Number'].notnull()]
unid_df_valid
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Number</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>BN001</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BN002</td>
    </tr>
    <tr>
      <th>2</th>
      <td>BN003</td>
    </tr>
    <tr>
      <th>3</th>
      <td>BN004</td>
    </tr>
    <tr>
      <th>4</th>
      <td>BN005</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>664</th>
      <td>BN665</td>
    </tr>
    <tr>
      <th>666</th>
      <td>BN667</td>
    </tr>
    <tr>
      <th>667</th>
      <td>BN668</td>
    </tr>
    <tr>
      <th>669</th>
      <td>BN670</td>
    </tr>
    <tr>
      <th>670</th>
      <td>BN671</td>
    </tr>
  </tbody>
</table>
<p>383 rows × 1 columns</p>
</div>




```python
unid_df_valid.to_excel('domestic_unidentified.xlsx', index = False)
```


```python

```
