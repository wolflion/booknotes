## ACM模式输入输出

+ 2023-10-29考试，反省了2个问题，一个是**太紧张，题目没有真正去读懂**，第二是**ACM输入输出问题**，导致调试这个花了些时间，2023-10-30就花了一个cubi整理一下，在日常过程中不断练习，**先知道怎么用，再去把脉络和细节搞清楚**
+ 具体措施是**每刷一道题，都自己模拟输入输出**，先刷50道这个量吧，练习到一写就对，不需要调试的地步（现在属于知道，也能调试，但是熟练度完全不够，**导致考试时很浪费时间**）

### 输入

#### 基础1：istringstream，getline()，cin，cin.get()

##### istringstream

+ **头文件是`#include<sstream>`**

```cpp
int main(){
    string str = "I love china";
    istringstream iss(str);
    while(iss>>str){
        cout<<str<<endl;
    }
    cout<<str;
}
```



#### 整型数组输入1：规定输入几个，空格分割的字符

+ vector数组未初始化大小时，只能用push_back方式插入数据  注意  不能vector<int> nums(n)这样默认前n个数为0，**这个是啥原因**？

```cpp
    //方法1：固定大小n   优先用方法1
    int n;
    cin >> n;
    vector<int> nums(n);
    for (int i = 0; i < n; ++i){
        cin >> nums[i];
    }
    //方法2：resize大小为n
    int n;
    cin >> n;
    vector<int> nums;
    nums.resize(n);
    for (int i = 0; i < n; ++i){
        cin >> nums[i];
    }
    //方法3：vector数组未初始化大小时，只能用push_back方式插入数据  注意  不能vector<int> nums(n)这样默认前n个数为0
    int n;
    cin >> n;
    vector<int> nums;
    for (int i = 0; i < n; ++i){
        int val;
        cin >> val;
        nums.push_back(val);
    }
```



#### 整型数组输入2：不规定输入几个，空格分隔的字符

+ 代码通过cin.get()从缓存中读取一个字节，这样就扩充了cin只能用空格和TAB两个作为分隔符。**想想这原理是啥**
+ 同时想一下，2者有啥区别，一个是C的，一个是CPP的吗？

```cpp
/*
示例：1 2 3
*/

int main(){
    //实现1
    vector<int> nums;
    int num;
    while(cin>>num){
        nums.push_back(num);
        if(getchar() == '\n')//与实现2的区别
            break;
    }
    
    //实现2
    vector<int> nums;
    int num;
    while(cin>>num){
        nums.push_back(num);
        if(cin.get() == '\n')
            break;
    }
}
```

#### 字符串输入1：一行字符串，空格隔开

+ 问题是：**读入一行的时候，为啥要用`while (cin >> str) `**，是不是**因为中间存在空格啊**，那可以使用`getline(cin,str)`吧，**但也用了`while (getline(cin, input))`
+ **这个要debug一下**

```cpp
/*
示例： daa ma yello
*/
int main() {
	string str;
	vector<string> strs;
	while (cin >> str) {
		strs.push_back(str);
		if (getchar() == '\n') {  //控制测试样例           
			for (auto& str : strs) {
				cout << "当前单词："<< str << " ";
			}
			cout << endl;
			strs.clear();
		}
	}
	return 0;
}

//实现2
int main() {
	string input;
	while (getline(cin, input)) {  //读取一行
		stringstream data(input);  //使用字符串流
		int num = 0, sum = 0;
		while (data >> num) {
			sum += num;
		}
		cout << sum << endl;
	}
	return 0;
}
```



#### 字符串输入2：给定**一行字符串**，每个字符串用**逗号**间隔

```cpp
int main() {
	string input;
	while (getline(cin, input)) {  //读取一行
        vector<string> strs;
        string str;
        stringstream ss(input);
        while(getline(ss, str,',')){//lionel，这里面用的是','是分隔符，肯定我是不会的
            strs.push_back(str);
        }
        sort(strs.begin(), strs.end());
		for (auto& str : strs) {
			cout << str << " ";
		}
		cout << endl;
    }
	return 0;
}


//实例2：另一种分隔逗号的方式，（这个比较符合我的写的网格，但经常会写错，没有真正弄懂，每一句）
struct Item {
	int prio;
	int idx;
};

void od_a10() {
	string str;  // 输入9,3,5
	getline(cin, str);

	//分割逗号，这个输入可能会用到
	vector<Item> vi;
	str += ",";  //lionel，我日常处理不对，可能是因为我少写了一个这句，导致最后处理不成
	int id = 0;
	while (str.find(",") != string::npos) {
		int idx = str.find(",");
		string t = str.substr(0, idx);
		str = str.substr(idx + 1);
		vi.push_back({ stoi(t), id++ });// 插入结构体
	}
}


//实现3：我没有验证过，只是看到过
void imp03(){
    int n;//输入有几个字符
    cin>>n;  //lionel，不输入这个，感觉没法做？
    string str;  // 输入9,3,5
	getline(cin, str);
    vector<int> nums(n);
    char sep;
    for(int i=0;i<n-1;i++){
        cin>>nums[i]>>sep;
    }
    cin>>nums[n-1];
}
```



#### 字符串输入到数组3：输入n行字符串，逗号分开，存到数组里

```cpp
/*
*  2,3   //这一行表示有n行，m列
   1,2,3
   4,5,6
*/
int main()
{
    //lionel，参考的原型是 只有一个n，就是2，直接用int n; cin>>n;即可
    int n,m;
    //lionel-error，我当时处理错误的是cin>>n>>m;这种导致都无法写入
    string line;
    cin>>line;
    int pos = line.find(',');
    n=stoi(line.substr(0,pos));
    m=stoi(line.substr(pos+1));
    
    //lionel，这里的思路是有了n后，直接输入一行，然后把','换成' '再利用istringstream的机制
    vector<vector<int>> vec(n,vector<int>(m));
    for (int i = 0; i < n;++i){//注意这里一定要有i控制，用while容易一直输入导致错误
        string s;
        cin >> s;
        replace(s.begin(), s.end(), ',', ' ');
        istringstream input(s);
        string tmp;
        for (int j = 0; j < m;++j){//内层循环也很重要
            input >> tmp;
            vec[i][j] = stoi(tmp);
        }
    }

    for (int i = 0; i < vec.size(); i++)
    {
        for (int j = 0; j < vec[0].size(); j++)
        {
            cout << "输出是" << vec[i][j] << endl;
        }
    }
    return 0;
}

```



### 输出

#### 输出1：“A,B,C”

```cpp
//方法1
if (!res.empty()) {
    string s = to_string(res[0]);  //先把第1个整出来，换成s;   【s要放在最外面，放在这也行】
	for (int i = 1; i < res.size(); i++){
		s += ",";
		s += to_string(res[i]);
	}
	cout<<s<<endl;
}

//方法2
string s;
for(int i=0;i<res.size();i++){
    if(i>0){
        s+=",";
    }
    s+=to_string(res[i]);  //这才是对的；lionel；这个地方为0的时候，先输出来，后再大于0了，再输入逗号，再输入后面的值
    //下面是错误示范
    #if 0
    s+=to_string(res[i]);
    if(i>0){
        s+=",";  //这样第0个的时候，与第1个合在一起了
    }
    #endif
}
```



### 做题过程中的实践

#### od48数组拼接

```cpp
void od48() {
	int len;
	cin >> len;
	int arr_count;
	cin >> arr_count;

	vector<vector<int>> input(arr_count);
	//while (arr_count--) {
	for(int i=0;i<arr_count;i++) { //换这种方式的原因呢，正好把第一行push进去，用while的话，有点颠倒
		string str;
		//getline(cin, str);//因为上面有了cin>>arr_count，这个地方会形成 空行
		//while (getline(cin, str)){}  //但最后怎么停，没想好
		cin >> str;//直接用这种
		replace(str.begin(), str.end(), ',', ' '); //lionel，这个地方，不要用""，会报错，replace应该是算法
		istringstream iss(str);
		string tmp;
		while (iss >> tmp) {
			input[i].push_back(stoi(tmp));  //lionel,如果push_back有报错，一般是类型不对导致的
		}
	}
}
```



#### od75模拟消息队列

```cpp
/*
输入：
2 22 1 11 4 44 5 55 3 33
1 7 2 3
*/

int od75(){
    string linep;
	string linec;
	//cin >> linep;
	//cin >> linec;  //问题1：习惯性的用cin了，因为含有空格，就必须要用getline()才能输入整个
	getline(cin, linep);
	getline(cin, linec);

	vector<pair<int, int>> produce, consumer;
	istringstream issp(linep);
	int num, content;
	while (issp >> num>>content) {  //问题2：就是之前只知道一个个输，不知道可以2组一起输入到pair里
		produce.push_back(make_pair(num, content));
	}
	istringstream issc(linec);
	int start, end;
	while (issc >> start >> end) {
		consumer.push_back(make_pair(start, end));
	}
}
```



### 参考

+ [ACM模式各种输入总结C++版](https://zhuanlan.zhihu.com/p/494535515)，*都消化了*
  + 大概2023-10-27看过，但熟练度不够，不是看一遍，就写得很自如的

#### 未看的

+ https://blog.csdn.net/qq_41575507/article/details/90936476
+ [getline()的用法详解](https://blog.csdn.net/pangyou3s/article/details/128814684)
+ [getline函数详解](https://www.jianshu.com/p/ef56ec7a0af6)
+ 基于stringstream+getline实现字符串分隔

遗留的是，C++正则要想一下怎么做？