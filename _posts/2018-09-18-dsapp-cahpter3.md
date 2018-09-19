---
title: 邓俊辉数据结构学习-3-栈
tags: 数据结构 栈 八皇后 表达式 迷宫 回朔 孪生栈
key: "dsapp-3-stack-20180918"
---

# 栈的学习

## 栈的应用场合
<!--more-->
1. 逆序输出
    * 输出次序与处理过程颠倒，递归深度和输出长度不易预知
    * 不是很理解 <br/> 
    * 实例：进制转换
    * 大致思路：对于进制转换，我们一般使用的都是长除法，因此要保存每次得到的余数，但是最后算下来
    新的进制的数，刚好和存储的时候是逆序的。因此利用栈输出即可。原因是栈是FiLo后进先出。

2. 递归嵌套
    * 具有自相似性的问题可递归描述，但分支位置和嵌套深度不固定
    * 实例：括号匹配，
    * 实例：栈混洗
    * 联系，n个括号构成的合理种类和n个元素栈混洗的数目是一致的。

3. 延迟缓存
    * 线性扫描算法模式中，在预读足够长之后，方能确定可处理的前缀。
    * 表达式求值
    * 求RPN, 中缀转后缀表达式
    * 这里核心的概念是优先级表的构造, 对于表达式求值来说，比如说输入10个数，我们并不能确定其全部的
    计算顺序，只能确定其部分比如， (1+2)*3-5/6....我们知道可以优先计算1+2是，然后可以计算乘法，接下来
    除法该不该计算取决于后面的符号。这种知道谁该先计算就是优先级表的概念。因为我们直到遇到)才可以去
    计算+法，此时我们必须将+先存储起来，这里就是使用栈进行村粗。
    * 求后缀表达式的核心和表达式求值是一个道理。

4. 栈式计算(完全不了解)
    * 基于栈结构的特定计算

解题思路： 应用充分性而不是必要性。 <br/> 
https://www.zhihu.com/question/30469121 <br/> 

举例： 我们家人很高，充分条件，我们家人，即当人是我们家人的时候，结论身高很高。 <br/> 
当满足条件A，就可以推出结论B <br/> 
结婚需要买房， 买房即使结婚的必要条件。<br/>  根据结论B，可以推出条件A，同时也可以推出C，结婚只是一个条件之一。<br/> 

## 栈的小结
在很多的算法中，我们往往需要构造一个辅助栈来帮帮助我们延迟缓存和逆序输出。对于逆序输出来说，这个没什么好说的，
就是要记得有这个逆序的思想。最为经典的我认为莫过于二叉树的逆层次遍历。完全利用倒置的思想。还有就是回朔法，也是
利用栈的这个逆序的思想，深度优先遍历算是回朔法的一种吧。<br/> 
递归嵌套，理解的不深<br/> 

## 栈的习题
### N皇后问题
```cpp
struct Queen 
{ 
public: 
    Queen () = default;
    Queen (int xx, int yy)
    : x(xx)
    , y(yy) {}
     
    bool operator==(const Queen &rvalue)
    {
        if(x == rvalue.x || 
           y == rvalue.y ||
           x + y == rvalue.x + rvalue.y ||
           y - x == rvalue.y - rvalue.x)
            return true;
        return false;
    }

    int x;
    int y;
}; 
std::ostream & operator<<(std::ostream  &os, const Queen q)
{
    os << q.x << "," << q.y ;
    return os;
} 

const int N = 10;
int chessboard[N][N];

// 整体思路
// 1 如果皇后和已放置皇后序列不冲突，则加入皇后序列, 行号+1 goto 2, 否则改变当前皇后列的位置 goto 1
// 2 每行放置一个皇后
// 3 如果在皇后没有放完的情况下，无法放置。则重新改变上一个皇后的列 goto 1。

void placeQueen()
{
    vector<Queen> s;
    Queen q(0, 0);
    while(s.size() < N)
    {
        while(count(s.begin(), s.end(), q) && q.y < N)
        {
            ++q.y;
        }
        if( q.y < N)
        {
            s.push_back(q);
            ++q.x;
            q.y = 0;
        }
        else{
            if(!s.size())
            {
                cout << "no solution" << endl ;return ;
            }
            q = s[s.size() - 1];
            s.pop_back();
            ++q.y;
        }
    }
    for(auto &a : s)
    {
        cout << a << endl;
    }
}
```

### 迷宫路径问题

```cpp
using std::stack; 
// 迷宫初始可用状态， 形成路径状态， 回朔状态， 墙
typedef enum{Available, Route, BaskTrack, Wall} Status;

// unknown 为未定方向，即在没有进行方向搜索之前的状态 
// 这个设置很棒，这样在转向下一个方向的时候，只要加一下就可以。
typedef enum{Unkonwn, East, South, West, North, Noway} ESWN; // 定义的方向

// 返回下一个方向
inline ESWN nextESWN (ESWN eswn) { return ESWN(eswn + 1);}

struct Cell{
    int x, y;
    Status status;
    ESWN ingoing, outgoing; 
    Cell() = default;
    Cell(int xx, int yy, Status s = Available, ESWN i = Unkonwn, ESWN o = Unkonwn )
    : x(xx), y(yy), status(s), ingoing(i), outgoing(o) {}
    bool operator==(const Cell &rvalue)
    {
        if(x == rvalue.x && y == rvalue.y)
            return true;
        return false;
    }
};

#define Maxsize 24

Cell laby[Maxsize][Maxsize];

int randLaby(Cell *&startCell, Cell *&goalCell)
{
    srand(time(NULL));
    int ret;
    int size = Maxsize/2 + rand()%(Maxsize/2);
    for(int i = 0; i < size; ++i)
    {
        for(int j = 0; j < size; ++ j)
        {
            laby[i][j] = Cell(i, j, Wall);
        }
    }
    for(int i = 1; i < size - 1; ++i)
    {
        for(int j = 1; j < size - 1; ++j)
        {
            ret = rand() % 4;
            if(ret)
            {
                // 3/4 设置为可用
                laby[i][j] = Cell(i,j, Available);
            }
        }
    }
    // 设置起始点 和 结束点
    startCell = &laby[rand() % (size - 2) + 1][rand() % (size - 2) + 1];
    startCell->status = Available;
    goalCell = &laby[rand() % (size - 2) + 1][rand() % (size - 2) + 1];
    goalCell->status = Available;
    return size;
}

inline Cell* neighbour(Cell *c)
{
    switch(c->outgoing)
    {
        case East :
            return &laby[c->x][c->y + 1];
        case South :
            return &laby[c->x + 1][c->y];
        case West :
            return &laby[c->x][c->y - 1];
        case North :
            return &laby[c->x - 1][c->y];
        default:
            exit(-1);
    }
    return nullptr;
}
inline Cell* advance(Cell *c)
{
    switch(c->outgoing)
    {
        case East:
            c = &laby[c->x][c->y+1];
            c->ingoing = East;
            break;
        case South:
            c = &laby[c->x+1][c-> y];
            c-> ingoing = South;
            break;
        case West:
            c = &laby[c->x][c->y-1];
            c-> ingoing = West;
            break;
        case North:
            c = &laby[c->x-1][c->y];
            c-> ingoing = North;
            break;
        default:
            exit(-1);
    }
    return c;
}

void display(int size, Cell start, Cell goal)
{
    system("clear");
    for(int i = 0; i < size; ++i)
    {
        for(int j = 0; j < size; ++j)
        {
            switch(laby[i][j].status)
            {
                case Route : 
                    printf("*");
                    break;
                case Wall:
                    printf("#");
                    break;
                case BaskTrack:
                    printf("x");
                    break;
                case Available:
                    printf(" ");
                    break;
                default :
                    printf("-");
            }
            if(laby[i][j] == start)
               printf("\bS");
            else if(laby[i][j] == goal)
               printf("\bE");
        }
        printf("\n");
    }
}
bool findPath(Cell *start, Cell *goal, int size)
{
    if(start->status != Available || goal->status != Available)
    {
       return true;
    }
    stack<Cell*> s;
    start->status = Route;
    s.push(start);     
    do{
        // 展示迷宫
        display(size, *start, *goal);
        getchar();
        Cell *c = s.top();
        // 开始试探
        if(c == goal)
            return true;        
        // 试探所有方向
        while((c-> outgoing = nextESWN(c-> outgoing)) < Noway)
        {
            // 判断这个方向是否有路
            if(Available == neighbour(c)-> status) break;
        }
        // 如果所有方向都不行，就回朔
        if(c-> outgoing >= Noway && !s.empty())
        {
            c-> status = BaskTrack;
            c = s.top();
            s.pop();
        }
        else if( c->outgoing >= Noway && s.size() <= 1)
        {
            printf("failed\n");
            exit(-1);
        } 
        else{
            s.push( c = advance(c));
            c-> outgoing = Unkonwn;
            c-> status = Route;
        }
    }while(!s.empty());
    return false;
}

```

哇，这里不得不佩服邓俊辉老师这个题的思路，非常容易理解，而且非常好阅读 <br/> 
其主要思路为: 而且关键的一点是要明白我们是对这个迷宫地图做文章<br/> 
1. 将start节点加入Path中。
2. 检查Path.top() 的四个方向，寻找一个可以走的方向。
    1. 如果没有，将Path.top()的状态设置为BackTrack(这样下一次检查就会pass这个方向)，然后Path.pop()
    2. 如果有，走向这个方向，并将当前节点的状态设置为Route，然后将当前节点加入到Path中
3. 如果Path不为空 goto 2。

因此根据这个思路可以改写成我自己的方式。首先是迷宫的初始化，墙设置为'#'，可以走的路设置为' ',
已经走过的路设置为'*'，回朔过不能走的路标记为'!'。那么我判断上下左右能不能走只要去判断这个二维
向量的值是否为' '就可以了。走到这一步，就将其加入路径设置为'*'带表我走过。如果刚才的上下左右都
不能走，将其标记为'!'，然后pop掉Path.top()，再重新取Path中的top来进行下一轮判断。代码如下。

```cpp
const int Maxsize = 24;
char laby[Maxsize][Maxsize];

using Point = pair<int, int>;
class Laby 
{ 
public:
public: 
    Laby () {   init();     }
    void init()
    {
        srand(time(NULL));
        size = Maxsize / 2 + rand() % (Maxsize / 2);
        start.first = rand() % (size - 2) + 1;
        start.second = rand() % (size - 2) + 1;
        goal.first = rand() % (size - 2) + 1;
        goal.second = rand() % (size - 2) + 1;
        for(int i = 0; i < size; ++i)
        {
            for(int j = 0; j < size; ++j)
            {
                laby[i][j] = '#';
            }
        }
        for(int i = 1; i < size - 1; ++i)
        {
            for(int j = 1; j < size - 1; ++j)
            {
                if(rand() %4)
                    laby[i][j] = ' ';
            }
        }
        laby[start.first][start.second] = ' ';
        laby[goal.first][goal.second] = ' ';
    }

    void display()
    {
        system("clear");
        for(int i = 0; i < size; ++i)
        {
            for(int j = 0; j < size; ++j)
            {
                if(i == goal.first && j == goal.second)
                    printf("G");
                else
                    printf("%c", laby[i][j]);
            }
            printf("\n");
        }
    }

    Point findAvailableDirection(Point p){
        for(int i = 0; i < 4; ++i)
        {
            if(laby[p.first + 1][p.second] == ' ')
                return std::make_pair(p.first + 1, p.second);
            if(laby[p.first - 1][p.second] == ' ')
                return std::make_pair(p.first - 1, p.second);
            if(laby[p.first][p.second + 1] == ' ')
                return std::make_pair(p.first , p.second + 1);
            if(laby[p.first][p.second - 1] == ' ')
                return std::make_pair(p.first , p.second - 1);
        }
        return std::make_pair(-1, -1);
    }

    bool findPath()
    {
        stack<Point> path;
        path.push(start);
        laby[start.first][start.second] = '*';
        do{
            display();
            getchar();
            Point tmp = path.top();
            if(tmp == goal)
                return true;
            // 首先要找当前点中一个可以的方向, 如果四个方向都不行就返回!
            auto ret = findAvailableDirection(tmp);
            if(ret == Point(-1,-1))
            {
                // 需要回朔
                laby[tmp.first][tmp.second] = '!';
                path.pop();
            }
            else{
                laby[tmp.first][tmp.second] = '*';
                path.push(ret);
            }
        }while(!path.empty());
        return false;
    }
     
private: 
    Point start;
    Point goal;
    int size;
}; 

bool operator==(Point a, Point b)
{
    if(a.first == b.first && a.second == b.second)
        return true;
    return false;
}
```

### 表达式求值

表达式求值基本上说只有一个核心问题，就是优先级表的问题。我们维护俩个栈，一个位操作符栈，
一个位操作数栈，在操作符栈的进栈过程中。我们要为了进行优先计算，让低优先级的操作符可以触发
一些列高优先级的操作符，从而进行运算。达到我们想要的优先级高先进行计算的目的
```cpp
#define N_OPTR 9
using Operator = enum {Add, Sub, Mul, Div, Pow, Fac, Lp, Rp, Eoe};

const char pri[N_OPTR][N_OPTR] = {
         /*add  sub  mul  div  pow  fac  lp   rp   eoe*/
/*add*/    {'>', '>', '<', '<', '<', '<', '<', '>', '>'},
/*sub*/    {'>', '>', '<', '<', '<', '<', '<', '>', '>'},
/*mul*/    {'>', '>', '>', '>', '<', '<', '<', '>', '>'},
/*div*/    {'>', '>', '>', '>', '<', '<', '<', '>', '>'},
/*pow*/    {'>', '>', '>', '>', '>', '<', '<', '>', '>'},
/*fac*/    {'>', '>', '>', '>', '>', '>', ' ', '>', '>'},
/*lp*/     {'<', '<', '<', '<', '<', '<', '<', '=', ' '},
/*rp*/     {' ', ' ', ' ', ' ', ' ', ' ', ' ', ' ', ' '},
/*eoe*/    {'<', '<', '<', '<', '<', '<', '<', ' ', '='},
 };

using namespace std;

void readNumber(char *&s, Stack<float> &opnd)
{
    opnd.push(static_cast<float>(*s - '0'));
    while(isdigit(*(++s)))
    {
        opnd.push(opnd.pop() * 10 + *s - '0');
    }
    if(*s != '.')
        return ;
    float fraction = 1;
    while(isdigit(*++s))
    {
        fraction = fraction / 10;
        opnd.push(opnd.pop() + (*s - '0') * fraction);
    }
}

int convert(char c)
{
    Operator op;
    switch(c)
    {
        case '+' : op = Add; break;
        case '-' : op = Sub; break;
        case '*' : op = Mul; break;
        case '/' : op = Div; break;
        case '^' : op = Pow; break;
        case '!' : op = Fac; break;
        case '(' : op = Lp;  break;
        case ')' : op = Rp;  break;
        case '$' : op = Eoe; break;
        case '#' : op = Eoe; break;
    }
    return op;
}

float eval(float op1, char op, float op2)
{
    switch(op)
    {
        case '+' :
            return op1 + op2;
        case '-' :
            return op1 - op2;
        case '*' :
            return op1 * op2;
        case '/' : 
            return op1 / op2;
        case '^' :
            return pow(op1, op2);
    }
    return -1;
}

float fac(float num) // 懒得写了，递归了
{
    if(num == 0)
        return 1;
    return num * fac(num - 1);
}
float eval(char op, float op1)
{
    if(op == '!')
        return fac(op1);
    return -1;
}

void process(char *&s, Stack<char> &optr, Stack<float> &opnd)
{
    int rop = convert(*s);
    int lop; 
    lop = convert(optr.top());
    switch(pri[lop][rop])
    {
        case '<' :
            optr.push(*s);
            ++s;
            break;
        case '>' :
            if(optr.top() == '!')
            {
                float op1 = opnd.pop();
                char opchar = optr.pop();
                opnd.push(eval(opchar, op1));
            }
            else{
                float op2 = opnd.pop(); float op1 = opnd.pop();
                char opchar = optr.pop();
                opnd.push(eval(op1, opchar, op2));
            }
            break;
        case '=' :
            optr.pop(); 
            ++s;
            break;
        case ' ' ://非法情况
            cout << "illegal expression" << endl;
            exit(-1);
    }
}

float evaluate(char * S)
{
    Stack<float> opnd; 
    Stack<char> optr;
    optr.push('#');
    char *idx = S;
    while(!optr.empty())
    {
        if(isdigit(*idx))
        {
            readNumber(idx, opnd);// 这个操作会导致idx恒定每次递增
        }
        else{
            process(idx, optr, opnd);// 这个操作不一定导致idx每次递增
        }
    }

    return opnd.top();
}

void testReadNumber()
{
    char str[100];
    scanf("%s", str);
    char *ss = str;
    Stack<float> opnd;
    readNumber(ss, opnd);
    cout << opnd.top() << endl;
}
int main()
{
    char str[1000];
    while(scanf("%s", str))
    {
        printf("%f\n", evaluate(str));
        memset(str, 0, sizeof(str));
    }
    return 0;
}
```

