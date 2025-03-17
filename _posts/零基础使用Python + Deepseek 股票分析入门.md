# 零基础使用Python + Deepseek 股票分析入门

## 提纲

1. 看完这篇文章你能获得什么？
   - 可以在自己的电脑上运行python程序
   - 可以自己通过AI的辅助，完成调用deepseek API接口的代码
   - 可以使用python抓取到想要的股票数据
2. 必备的软件和资源
   - 编译器VScode或cursor，这是运行python程序的工具
   - tushare或akshare，这是抓取股票信息的数据源头
   - deepseek API接口的token，这是使用deepseek的钥匙

3. 教程的详细步骤
   - 安装vscode
   - 安装python
   - 如何抓取股票信息数据
   - 如何调用deepseek API接口分析抓取到的数据

## 教程

### 1.VScode安装

（1）进入vscode官方网站下载客户端 <https://code.visualstudio.com>

注意区分版本，我的电脑是Mac os，所以就下载macOS的，如果使用的是windows系统，就下载windows版本的。

![image-20250317092834797](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250317092840190.png)

### 2.安装python

（1）Windows 安装 Python

1.下载 Python 安装包

打开官网 [Python 官网](https://www.python.org/)

点击 Downloads，选择 Windows，下载最新的 Python 3.x安装包（.exe 文件）。

2.运行安装程序

双击下载的 .exe 文件，启动安装程序。

务必勾选 “Add Python to PATH”（添加到系统环境变量）。

点击 Install Now 进行默认安装。

3.验证安装是否成功

打开 命令提示符（CMD）或 PowerShell，输入：

```
python --version
```

如果能看到类似 Python 3.x.x，说明安装成功！🎉

（2）macOS 安装 Python

从 Python 官网下载（适合不想装 Homebrew 的用户）

1. 打开 [Python 官网](https://www.python.org/)

2. 选择 Downloads> Mac OS，下载 .pkg 安装包

3. 双击安装，完成后在终端输入：

```
python3 --version
```

看到 Python 3.x.x 即可。

### 3.抓取股票信息

如果我们已经安装好了编译器（以下以vscode为例）和python程序，那么我们就可以开始写代码了。

（1）首先创建项目文件夹

![image-20250317100027117](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250317100027170.png)

我创建了一个test的文件夹作为项目的存放位置，与该项目的依赖包、代码文件都在这里存放。

（2）打开vscode，依次点击“open”，然后选择刚刚创建好的文件夹（我创建的叫test），如下图就可以开始写代码了。

![image-20250317100304378](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250317100304418.png)

（3）创建第一个pythpn程序

我们在左侧区域蓝色的框框里，点击右键，选择new file，创建一个test.py，一定要注意python程序的后缀名是“py”

![image-20250317100359878](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250317100359915.png)

（4）开始写程序

这里我们有两个选项，一是自己写，二是借助AI工具写。这里我先以自己写代码为例子，主要是帮助大家理解代码的逻辑，后续熟悉了以后可以借助AI工具完成代码。

**抓取股票信息**

我们从tushare官网去抓数据，就需要去它的官网看看，它给我们提供了哪些接口。

![image-20250317101039276](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250317101039314.png)

进入tushare的官网可以看到，它给我们提供了这么多数据接口，以日线行情为例子，![image-20250317101205632](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250317101205675.png)

我们可以看到，使用以下这个接口，就可以返回000001这只股票在20180701-20180718的日线行情数据。

```python
df = pro.daily(ts_code='000001.SZ', start_date='20180701', end_date='20180718')
```

那我们在自己的vscode里面去尝试一下。

![image-20250317101640162](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250317101640210.png)

```
import tushare as ts

# 设置token
ts.set_token('输入你自己的token')
pro = ts.pro_api()
# 获取数据
df = pro.daily(ts_code='000001.SZ', start_date='20180701', end_date='20180718')
# 打印输出
print(df)
```

**调用deepseek进行分析**

先使用命令pip install openai 安装AI的包

![image-20250317102204860](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250317102204905.png)

在程序里配置AI的api_key和base_url

![image-20250317103702252](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250317103702297.png)

写一个函数调用deepseek api，并把股票信息传给AI

![image-20250317103837466](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250317103837518.png)

最后调用函数即可：

![image-20250317104155381](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250317104155434.png)

点击运行后的结果（大概需要1分钟的等待时间）：

![image-20250317104102958](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250317104103008.png)

---

### **关键走势分析**
1. **大幅下跌后反弹（7月2日-7月9日）**  
   - **7月2日**：股价暴跌5.28%，成交量激增，显示恐慌性抛售。
   - **7月3日-6日**：价格在低位震荡，成交量逐步减少，卖压减弱后出现小幅反弹（7月6日涨0.7%）。
   - **7月9日**：放量大涨4.27%，成交量达140万手，形成短期底部突破，可能受利好消息驱动。

2. **冲高回落与阻力显现（7月10日-12日）**  
   - 价格在9.03附近遇阻（7月9日高点），随后小幅回调，但7月12日再次放量上涨2.78%，试探前高未果，显示上方抛压较强。

3. **上涨乏力与短期回调（7月13日-18日）**  
   - **成交量萎缩**：7月13日后成交量持续下降，市场参与度降低。
   - **高点下移**：7月12日高点8.97 → 7月16日8.90 → 7月18日8.85，形成下降趋势线，表明多头动能不足。
   - **逐步阴跌**：收盘价从8.88（7月12日）跌至8.70（7月18日），确认短期回调趋势。

---

### **技术信号**
- **阻力与支撑**  
  - **阻力位**：9.03（7月9日高点）→ 8.97（7月12日）→ 8.90（7月16日），上方压力逐步下移。
  - **支撑位**：8.60-8.66（7月5日-6日低点），若跌破可能进一步下探8.45。
  
- **量价背离**  
  价格反弹时成交量未持续放大（如7月12日后缩量），暗示上涨缺乏资金支持。

---

### **短期走势预测**
- **看跌信号**：近期高点下移、成交量萎缩、阴跌形态，预示短期内可能继续回调。
- **关键观察点**：  
  - 若股价跌破8.60支撑位，可能加速下跌。  
  - 若放量突破8.85-8.90阻力区，则可能重启上涨趋势。

---

### **操作建议**
- **谨慎持有**：短期风险偏向下行，可考虑逢高减仓或设置止损（如8.60）。  
- **观望信号**：等待放量突破阻力或企稳支撑后再做决策。

---

**注**：以上分析基于历史数据，实际走势需结合实时市场环境及基本面因素综合判断。



## 总结

完整源代码：

```
import tushare as ts
from openai import OpenAI

# 设置token
ts.set_token('你自己的token')
pro = ts.pro_api()
# 获取数据
df = pro.daily(ts_code='000001.SZ', start_date='20180701', end_date='20180718')
# 打印输出
print(df)


# 内置 OpenAI API 配置
DEEPSEEK_API_KEY = "你自己的key" 
client = OpenAI(
    api_key = DEEPSEEK_API_KEY,
    base_url = "https://ark.cn-beijing.volces.com/api/v3")

# AI分析分笔数据
def analysis_fenbi(fenbi_data):
    try:
        # 构造 Deepseek 提示词
        prompt = f"""
        分析以下分笔数据，判断股票走势：
        {fenbi_data}
        """
        # 调用 Deepseek API 进行分析
        response = client.chat.completions.create(
            model="ep-20250217151011-nmg5k",
            messages=[{"role": "user", "content": prompt}]
        )
        # 获取分析结果
        response_content = response.choices[0].message.content.strip()
        return response_content
    
    except Exception as e:
        return f"AI 分析生成失败: {str(e)}"

# 调用函数完成分析
if __name__ == "__main__":
    # 分析分笔数据
    fenbi_data = df.to_string()
    result = analysis_fenbi(fenbi_data)
    print(result)


```

这只是一个最基础的使用deepseek分析股票交易数据的案例，理解逻辑之后，可以灵活运用。后续我将继续分享更多的使用案例。包括自己在交易过程中使用的AI分析工具。

如果您觉得内容对您有帮助，请帮忙点个免费的关注，激励我继续创作更多内容。谢谢！