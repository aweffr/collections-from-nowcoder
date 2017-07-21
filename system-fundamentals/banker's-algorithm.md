# 银行家算法
*转载: http://www.cnblogs.com/hxsyl/p/3236301.html*
## 概念引入

        银行家算法（ banker's algorithm ）由 Dijkstra于1965提出，关键是将死锁的问题演示为一个银行家贷款的模型，由于能用于银行系统的现金贷款而出名。一个银行家向一群客户发放信用卡，每个客户有不同的信用额度。每个客户可以提出信用额度内的任意额度的请求，直到额度用完后再一次性还款。银行家承诺每个客户最终都能获得自己需要的额度。所谓“最终”，是说银行家可以先挂起某个额度请求较大的客户的请求，优先满足小额度的请求，等小额度的请求还款后，再处理挂起的请求。这样，资金能够永远流通。所以银行家算法其核心是：保证银行家系统的资源数至少不小于一个客户的所需要的资源数。

        银行家算法是一种最有代表性的避免死锁的算法。在避免死锁方法中允许进程动态地申请资源，但银行家算法在系统在进行资源分配之前(并不是真的不分配，这样就没法做了，只不过是试探性分配，不满足的话再恢复)，应先计算此次分配资源的安全性，若分配不会导致系统进入不安全状态，则分配，否则等待。为实现银行家算法，系统必须设置若干数据结构。要解释银行家算法，必须先解释操作系统安全状态和不安全状态。安全序列是指存在一个进程序列{P1，…，Pn}是安全的，不会死锁(至少两个线程占有某资源A，但是都不满足，剩余的资源A分配给谁仍然无法满足)，安全状态如果存在一个由系统中所有进程构成的安全序列P1，…，Pn，则系统处于安全状态，安全状态一定是没有死锁发生；不安全状态不存在一个安全序列，不安全状态不一定导致死锁。

        本算法在理论上是出色的，能非常有效地避免死锁，但从某种意义上说，它缺乏实用价值，因为很少有进程能够在运行前就知道其所需资源的最大值，且进程数也不是固定的，往往在不断地变化（如新用户登录或退出），况且原来可用的资源也可能突然间变成不可用（如打印机、磁带机可能被损坏）。
##  算法原理

        银行家算法的基本思想是分配资源之前，判断系统是否是安全的；若是，才分配。每
        分配一次资源就测试一次是否安全，不是资源全部就位后才测试，注意理解
        checkError函数的循环顺序。

        我们可以把操作系统看作是银行家，操作系统管理的资源相当于银行家管理的资金，
        进程向操作系统请求分配资源相当于用户向银行家贷款。 
        为保证资金的安全，银行家规定：

        当一个顾客对资金的最大需求量不超过银行家现有的资金时就可接纳该顾客(试探性分
        配)
        顾客可以分期贷款，但贷款的总数不能超过最大需求量(可能一次并不能满足所需要的
        全部资源)
        当银行家现有的资金不能满足顾客尚需的贷款数额时，对顾客的贷款可推迟支付，但
        总能使顾客在有限的时间里得到贷款(不存在死锁)
        当顾客得到所需的全部资金后，一定能在有限的时间里归还所有的资金(运行后释放)
        操作系统按照银行家制定的规则为进程分配资源，当进程首次申请资源时，要测试该
        进程对资源的最大需求量，如果系统现存的资源可以满足它的最大需求量则按当前的
        申请量分配资源，否则就推迟分
        配。当进程在执行中继续申请资源时，先测试该进程本次申请的资源数是否超过了该
        资源所剩余的总量。若超过则拒绝分配资源，若能存在安全状态，则按当前的申请量
        分配资源，否则也要推迟分配。

## Java代码
```java
import java.util.Arrays;
import javax.swing.JOptionPane;

public class Banker {

  /*
   * 资源向量必须全部设置成static，因为可能
   * 同一个线程多次输入才满足条件
   */
  //每个线程需要的资源数
  static int max[][] = { { 7, 5, 3 }, { 3, 2, 2 }, { 9, 0, 2 },
    { 2, 2, 2 }, { 4, 3, 3 } };
  //系统可用资源数
  static int avaliable[] = {10,5,7};
  //已经分配资源
  static int allocation[][] = { { 0, 0, 0 }, { 0, 0, 0 }, { 0, 0, 0 },
    { 0, 0, 0 }, { 0, 0, 0 } };
  //每个进程还需要的资源数,初试一个资源也没分配；实际上应该等于max-avaliable
  static int need[][] = Arrays.copyOf(max,max.length);
  //每次申请的资源数
  static int request[] = { 0, 0, 0 };
  //NUM个线程，N种资源
  static final int NUM = 5, N = 3;
  static Function function = new Function();
  
  public static void main(String[] args) {
    JOptionPane jpane = new JOptionPane();
    
    //是否进行模拟标志，没有布尔，因为从JOpotionpane输入
    int flag = 1;
    
    while(1==flag) {
      /*
       * 用与判断线程号是否合法
       * 需要放在while内部，防止下次继续模拟时i还是上次输入的
       */
      int i = -1;
      while(i<0||i>=NUM) {
        String str = jpane.showInputDialog("输入申请资源的线程号(0到4)：");
        i = Integer.parseInt(str);
        if(i<0||i>=NUM) {
          JOptionPane.showMessageDialog(jpane, "输入的线程号不合法！！！");
        }
      }
      //资源输入有效性标志
      boolean tag = true; 
      for(int j=0; j<N; j++) {
        String str = jpane.showInputDialog("输入线程"+i+"所申请的资源"+j+"数目：");
        request[j] = Integer.parseInt(str);
        //有效性检查
        if(request[j]>need[i][j]) {
          JOptionPane.showMessageDialog(jpane, "输入的资源数大于需要资源数！！！");
          tag = false;
          break;
        }else {
          if(request[j]>avaliable[j]) {
            JOptionPane.showMessageDialog(jpane, "输入的资源数大于可用资源数！！！");
            tag = false;
            break;
          }
        }
      }
      //是否存在安全序列
      boolean vis = true;
      if(tag) {
        function.allocateK(i);
        vis = function.checkError(i);
        if(false==vis) {
          //上面调用了allocateK，所以不仅需要释放，还需要恢复
          function.freeKAndRestore(i);
        }else {
          //测试是否全部资源到位
          boolean f = function.checkRun(i);
          if(true==f) {
            JOptionPane.showMessageDialog(jpane
                ,"进程"+i+"全部资源到位！！！"+"\n"+"即将释放所占用资源");
            function.freeKNotRestore(i);
          }
        }
      }else {
        //实际上没必要清空，因为该数组是输入的，只为了展示一种良好习惯
        Arrays.fill(request,0);
      }
      String str = JOptionPane.showInputDialog("是否继续模拟(1表示是，0退出)？");
      flag = Integer.parseInt(str);
    }
  }
}

class Function {
  /*
   * 实际上完全是静态的，没必要新new一个Banker
   */
  Banker banker = new Banker();
  //为线程k分配资源
  public void allocateK(int k) {
    for(int i=0; i<banker.N; i++) {
      banker.avaliable[i] -= banker.request[i];
      banker.need[k][i] -= banker.request[i];
      banker.allocation[k][i] += banker.request[i];
    }
  }
  public boolean checkError(int i) {
    int work = 0;
    //存储所有线程是否安全
    boolean[] finish = new boolean[banker.NUM];
    Arrays.fill(finish,false);
    //存储一个安全序列
    int temp[] = new int[banker.NUM];
    Arrays.fill(temp,0);
    //temp数组下标
    int t = 0;
    
    //线程号参数是i
    for(int j=0; j<banker.N; j++) {
      work = banker.avaliable[j];
      int k = i;
      
      while(k<banker.NUM) {
        if(finish[k]==false&&work>=banker.need[k][j]) {
          /*
           *  注意不是max数组，因为此时线程k
           *  所需资源不一定完全就位
           *  加的是allocation，因为进行此项检查前先试探性地
           *  分配给线程k资源了
           */
          //满足该线程，回收该项资源，看是否满足其它线程
          work += banker.allocation[k][j];
          finish[k] = true;
          temp[t++] = k;
          k = 0;
          
        }else {
          k++;
        }
      }
      //和while平级
      for(int p=0; p<banker.NUM; p++) {
        if(finish[p]==false) {
          return false;
        }
      }
    }
    return true;
  }
  //释放线程k所占用资源并恢复
  public void freeKAndRestore(int k) {
    for(int i=0; i<banker.N; i++) {
      banker.avaliable[i] += banker.request[i];
      banker.need[k][i] += banker.request[i];
      banker.allocation[k][i] -= banker.request[i];
    }
  }
  //仅仅释放线程k所占用资源，仅在某线程全部得到资源运行后才调用
  public void freeKNotRestore(int k) {
    for(int i=0; i<banker.N; i++) {
      banker.avaliable[i] = banker.avaliable[i] + banker.allocation[k][i];
    }
  }
  //三种资源是否全部到位
  public boolean checkRun(int k) {
    int n = 0;
    for(int i=0; i<banker.N; i++) {
      if (banker.need[k][i] == 0)
        n++;
    }
    if (n == 3)
      return true;
    else
      return false;
  }
}
```