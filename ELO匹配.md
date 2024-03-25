# BETA - MATCH 天梯匹配系统设计稿(DeskStand V4 Version)

> 该匹配系统是基于玩家的技术水平和等级来进行配对。玩家在系统中分配一个排名，根据排名来匹配同等级的玩家进行比赛。匹配系统还会考虑玩家的地理位置和网络条件，以确保游戏的公平性和流畅性。匹配系统还会对玩家的胜利和失败次数进行统计，以便更好地匹配玩家和提供更具挑战性的对手。

## 系统实现清单

CSGO匹配系统需要实现以下功能：

1. 玩家排队匹配：玩家队伍可以选择在特定地图或模式下排队匹配，系统会自动根据队伍情况决定是否平衡人数，参加匹配。
2. 匹配算法：系统需要设计一个有效的匹配算法，以确保匹配到的队伍在技术水平、地理位置等方面能够平衡。
3. 排位赛匹配：系统可以根据玩家的胜负记录和技术水平进行排位赛匹配，以确保比赛的公平性。
4. 匹配等待时间提示：系统会提示玩家当前匹配等待时间，以便玩家可以选择等待或取消匹配。
5. 匹配结果反馈：系统会向玩家反馈匹配结果，包括匹配到的队伍信息、比赛时间等。
7. 匹配历史记录：系统会记录玩家的匹配历史，包括胜负记录、技术水平变化等，以便玩家了解自己的表现和进步情况。



## 类设计

分为：枚举完成系统所需要的类、类之间的关系、类具体的定义（包括封装的方法声明、对象）

#### 枚举类

1. Player类：表示玩家，包括玩家ID、技术水平、胜负记录等属性，以及与玩家相关的操作方法，如排队匹配、取消匹配等。
2. Match类：表示比赛，包括比赛ID、比赛地图、比赛模式、参与队伍等属性，以及与比赛相关的操作方法，如匹配算法、匹配结果反馈等。
3. Team类：表示队伍，包括队伍ID、队伍成员、队伍技术水平等属性，以及与队伍相关的操作方法，如队伍平衡、队伍匹配等。
4. MatchmakingSystem类：表示匹配系统，包括玩家排队匹配、排位赛匹配、自定义匹配选项等功能的实现，以及匹配历史记录的管理。



#### 类关系

- **MatchmakingSystem** 是整个系统的核心类，管辖所有的**Match**类对象集合，可**Match**比赛对象数组进行增、删、改、查等操作。

- **Match**类与**Team**类有关联关系，一个Match对象可以包含多个Team对象参与比赛。
- **Team**类与**Match**类有单一关联关系，*一个**Team**对象在匹配成功中只能绑定一个**Match**对象。* 不可出现一次匹配请求，绑定多个Match对象问题。
- **Player**类与**MatchmakingSystem**类有层级关联关系，玩家可以通过**MatchmakingSystem**类进行排队匹配、取消匹配。**MatchmakingSystem**类在收到**Player**对象发起的对于**Match**类的请求时，根据规则对**Match**对象进行操作。

#### 类定义

Player 类定义：

```c#
// Player类定义
public class Player
{
    public int UID { get; set; } // Player ID
    public string UserName {get; set;} // Player Name
    public bool ActionAntiCheat {get; set;} // Player Whether Runing AntiCheat  
    public PlayerInfo PInfo{get; set;} // Player Info
    public PlayerHistorMatchData PHMD {get; set;} // Player Histor Match Data
    public PlayerDevice PDevice {get; set;} // Player Device Info
}

public class PlayerInfo 
{
	public int PlayTime {get; set;} // 游戏时长
	public int ReputationScore {get; set;}//信誉分
	public double KD {get; set;}; // KD 比
	public int Kills { get; set; } // 击杀
    public int Deaths { get; set; } // 死亡
    public int Assists { get; set; } // 助攻
    public int Headshots { get; set; } // 暴击
}

public class PlayerHistorMatchData 
{
    public int SkillLevel { get; set; }// 匹配系统中的排名
    public int WinCount { get; set; }// 胜利次数
    public int LossCount { get; set; }// 失败次数
}

public class PlayerDevice
{
    public string DeviceType { get; set; } // 设备类型
    public string DeviceOS { get; set; } // 设备操作系统
    public string DeviceModel { get; set; } // 设备型号
    public string DeviceIPAddress { get; set; } // 设备IP地址
    public int Ping { get; set; } // 设备网络延迟
    public string CPUInfo { get; set; } // CPU信息
    public string GPUInfo { get; set; } // GPU信息
    public string RAMInfo { get; set; } // 内存信息
    public string StorageInfo { get; set; } // 存储信息
}

public class PlayerBanData
{
    public bool IsBanned { get; set; } // 是否被禁赛
    public DateTime BanStartTime { get; set; } // 禁赛开始时间
    public DateTime BanEndTime { get; set; } // 禁赛结束时间
    public string BanReason { get; set; } // 禁赛原因
}
```

Team 类定义：

```csharp
public class Team
{
    public string TeamID { get; set; } // 队伍ID
    public List<Player> TeamMembers { get; set; } // 队伍成员列表
    public int TeamSkillLevel { get; set; } // 队伍技术水平
    public int TeamRank { get; set; } // 队伍排名
    public int TeamAveragePing { get; set; } // 队伍平均网络延迟
    public string TeamMode {get; set;} // 队伍匹配模式
    public Team_MatchInfo T_MatchInfo { get; set;}
}

public class Team_MatchInfo
{
    public int TeamInMatchSerial { get; set;} // 队伍在匹配系统中的序号
    public string Expect_Success_Time { get; set;} // 预计匹配成功时间
    public string Join_Match_Time { get; set;} // 加入匹配时间
}
```



Match 类定义 ：

```c#
public class Match 
{
   public int MatchID { get; set; } // 比赛ID
    public string Map { get; set; } // 比赛地图
    public string Mode { get; set; } // 比赛模式
    public List<Team> ParticipatingTeams { get; set; } // 参与比赛的队伍列表
    public DateTime MatchStartTime { get; set; } // 比赛开始时间
    public DateTime MatchEndTime { get; set; } // 比赛结束时间
    public string MatchResult { get; set; } // 比赛结果
    public Server Match_Server { get; set;} // 比赛服务器信息
}

public class Server 
{
    public string ServerID { get; set; } // 服务器ID
    public string ServerName { get; set; } // 服务器名称
    public string IP_Address { get; set; } // 服务器IP地址
    public string Port { get; set; } // 服务器Port
    public string ServerLocation { get; set; } // 服务器地理位置
}
```

MatchmakingSystem类定义：

```c#
public class MatchmakingSystem
{
    public List<Player> Players { get; set; } // 玩家队列
    public List<Team> TeamsQueue { get; set; } // 队伍排队队列
}
```

## 功能实现设想

##### 玩家排队匹配

> 玩家队伍可以选择在特定地图或模式下排队匹配，系统会自动根据队伍情况决定是否平衡人数，参加匹配。

实现设想：匹配系统实现队伍与队伍之间的匹配。

玩家在UI中的操作流程应为：准备->开始匹配

映射UI到数据处理为：

1. 准备 -> 在匹配系统中添加到玩家队列中；
2. 开始匹配 ->
   1.  将当前队伍信息进行匹配前校验，如人数校验，检测玩家队列中是否有对应的成员存在。
   2.  校验通过添加队伍至匹配队列中
   3.  更新队伍的**Team_MatchInfo**信息