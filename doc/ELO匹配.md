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
    public bool User
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

## 匹配系统核心流程

#### 基础阶段流程：

1. ##### 用户每发起一次准备请求，检测：

   1. 用户是否被禁赛
   2. 用户是否开启反作弊系统
   3. ....
   4. 用户是否在准备队列

2. ##### 用户队伍发起匹配请求：

   1. 成员确认： 核对队伍 Team.TeamMembers 的 成员是否均在准备队列中，若有成员不在，则返回 **<u>成员未准备事件</u>** 。
   2. 模式分类处理：Team.TeamMode 将判定队伍添加进入对应的匹配池中。

3. ##### 匹配池处理：

   1. 平衡人数：

      若 Team.TeamMembers.leng 长度小于 匹配模式所规定的队伍人数 ，则将队伍加进挂起等待队列。等待符合添加的新队伍合   并为新 Team 对象。人数平衡加入匹配池中，寻找对手。

       新 Team 对象合并要求：两支队伍

      1. 人数平衡： Team.TeamMembers.leng 相加必须 ( 等于 或 *小于[ 争论项，此条件会使搜索时间复杂度↓，但会增加Player属性：已确保队伍能出现恢复重组]*) 匹配模式所规定的队伍人数。
      2. 实力平衡：确保 Team.TeamAveragePing 和 Team.TeamRank 在可接纳范围内。

   2. 寻找对手队伍：

        本质上也是平衡人数，这里考虑继承平衡人数方法，并将平衡人数条件修改为等于模式所需要的总人数。

      ```bash
      两只队伍.TeamMembers.length == 模式所需要的总人数
      ```

   3. 部分模式要求：两只队伍倒计时投票决定Match中的某些属性值

         例子情景：两支队伍，共10人。有地图列表共11张，要求15s时间内，Team1 下 5名成员以投票的方式，投出5张地图。同样，Team2 将投出剩下5张。至此，地图池仅剩一张，作为 Match.Map 的值。

        注意：若 Team1 进行 15s 倒计时投票，Team2 将等待 Team1 投票结果。

        实现参考：

        1. 创建方法VoteForMap，接收2支队伍组成的Team对象数组 | 地图列表，返回最后的结果
      

         ```c#
         // 接收2支队伍的Team对象，返回最后的结果
         public string VoteForMap(Team[] teams, List<string> mapPool)
         {
                   
         }
         ```

      2. 创建循环，依次向 Team A，Team B 推送 Ban 图流程，使其进入 Ban 图状态。

         ```c#
         public string VoteForMap(Team[] teams, List<string> mapPool)
         {
             for(int i = 0; i < teams.Length; i++)
             {
                 // 15s 倒计时模拟，每秒向 teams 子对象发送 更新的地图列表
                for(int j = 0; j < 15; j++)
                {
                     await UpdataBanMapList(teams[i], mapPool); 
                     await Task.Delay(1000); // 等待1秒
                }
             }      
         }
         
         public string[] UpdataBanMapList(Team BanMapTeam)
         {
             
         }
         
         
         ```
