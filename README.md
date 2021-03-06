# 南海清音古琴社自动化选课后台程序规划

问卷星制作问卷->导出为xlsx文件,作为原始数据(按序号导出)->数据清洗,计算班级频数(说明见3.3), 输出处理后的数据->排课->输出课表为json文件(方便后续处理,如后期建立图形界面)

编程语言: python

## 1.问卷示例

见二维码所指问卷:

<img src="./statistics/img/qrcode.jpg" style="zoom:50%;" />

## 2.输入数据示例

![]()

J,K,L和M列为第四题(所选的曲目课程课程)的四个选项, 0代表没有选此项, 1反之. ~~N,O,P,Q,R和S列为第五题(学过的曲子)的选项.~~

## 3.排课逻辑

基于自由度, 问卷填写时间, 曲目合适程度, 班级频数进行排课.

### 3.1.曲目是否适合

用于判断某曲目是否适合某人学习~~以及合适程度~~

~~逻辑: 首先对所有曲目进行难度分级, 如1,2,3 ... ,数字越大代表越难,等级记为L. 然后将某人所学曲目中等级最高的曲目作为基数B. 合适程度D:~~

![](./statistics/img/mylatex20210224_004912.svg)

~~0 代表不适合, 除0外数字越小越适合.~~

~~例如总共有五个课程a,b,c,d,e, 他们的难度等级为L(a)=1, L(b)=2, L(c)=3, L(d)=4, L(e)=5. 同学A会的最难的曲目为2难度,即B=2. 再此情况下D(a)=3, D(b)=2, D(c)=1, D(d)=2, D(e)=0 为不推荐学习. c课程最合适, 其次是b和d, 然后是a.~~

首先对所有曲目进行难度分级,难度记录即为所要求的至少会的曲目数量,统计某人会的曲目数量,若大于所需要求,则可以选该曲目.

### 3.2.自由度

自由度分为个人自由度与同修自由度(一般只存在于开指班选课)

个人自由度: 当没有选同修时, 所选择的课程数量即为个人自由度. 如A同学选了课程一和课程二且没有选择同修,其个人自由度为2.

同修自由度: 当两人互相选择对方为同修时, 双方会被绑定为一对, 双方所选的重叠的课程的数目为同修自由度. 如A同学选了课程一与二, 且选了B为同修, B同学选了课程二与三, 且选了A为同修. A与B会被绑定为一对, 且其同修自由度为1.

### 3.3.班级频数

计算过程: 

1. 筛除每个人选的课中不适合学习的课程,若全不适合,沟通修改选课.
2. 统计每个课程被选次数, 此次数即为班级频数.

### 3.4.排课过程

情况一,开指班排课: 

1. 计算班级频数
1. 找出所有同修对, 按同修自由度从低到高与问卷填写时间由早到晚(自由度优先)为他们安排课程. 若某一同修对有一个以上的候选课程, 则优先分配到频数低的班级, 若所有侯选班频数相等,则随机分配.
2. 对所有其他同学, 按个人自由度从低到高与问卷填写时间由早到晚(自由度优先)为他们安排课程. 若某一同学有一个以上的候选课程, 则优先分配到频数低的班级, 若所有侯选班频数相等,则随机分配.
3. 若排课结果极度不平衡, 如某些班人特少, 某些特多, 则尝试人工换课. 若经常有此情况发生后续版本会考虑加入此功能.

情况二,非开指课程:

1. 计算班级频数
2. 对所有同学按个人自由度与问卷填写时间由早到晚(自由度优先)从低到高为他们安排课程. 若某一同学有一个以上的候选课程,则优先分入频数低的班, 若频数相等,则随机分配.
3. 若排课结果极度不平衡, 如某些班人特少, 某些特多, 则尝试人工换课. 若经常有此情况发生后续版本会考虑加入此功能.

## 4.待定点

2. 要不要为非开指班加入同修机制(按我的理解,同修机制应该只在10人以上的班存在,非开指班一般没有十人? 2020年的问卷里没有给非开指班同修选项)
3. 其他的任何问题

## 5.面向对象设计

程序结构设计

### 5.1.course类(course.py)

属性Property:

1. 课程名(name: name, type: str)
2. 时间(name: time, type: str, example: Monday|14:00-15:00)
3. 难度(name: difficulty , type: int)
4. 学生(name: students, type: list)
5. 同修对(name: pairs, type: list)
6. 当前学生数量(name: current_num, type: int)
7. 最大学生数量(name: max_num: , type: int, -1 代表无限制)

方法method:

1. 添加学生(def add_student(student))
2. 添加同修对(def add_pair(pair))
3. ~~判断是否满员(def if_full(), return Boolean)~~

### 5.2.pair类(pair.py)

属性Property:

1. 第一人名字(name: name1, type: str)
2. 第一人学号(name: id1, type: str)
3. 第二人名字(name: name2, type: str)
4. 第二人学号(name: id2, type: str)
5. 候选课程(name: candidate_class, type: list)
6. 填写时间(name: finish_time, type: str)
7. 自由度(name: DF, type: int)

### 5.3.student类(student.py)

属性Property:

1. 姓名(name: name, type: str)
2. 学号(name: id, type: str)
3. 候选课程(name: candidate_class, type: list)
4. 填写时间(name: finish_time, type: str)
5. 自由度(name: DF, type: int)

### 5.4.arranger类(arranger.py)

属性Property:

1. file(name: xlsx_file_path, type: str)

2. class_difficulty_list(name: txt_file_path, type: str)

   example: 

   file name: class_difficulty_list

   ```

   class1 2

   class2 3

   class3 4

   ```

method:

1. 生成标准data frame: generate_DF(name_index, id_index, class_index, learned_index, pair_id_index, time_index)
2. 计算班级频数: calculate_frequency()
3. 开始排课: start

