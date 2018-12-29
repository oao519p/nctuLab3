# Route Configuration

This repository is a lab for NCTU course "Introduction to Computer Networks 2018".

---
## Abstract

In this lab, we are going to write a Python program with Ryu SDN framework to build a simple software-defined network and compare the different between two forwarding rules.

---
## Objectives

1. Learn how to build a simple software-defined networking with Ryu SDN framework
2. Learn how to add forwarding rule into each OpenFlow switch

---
## Execution

> TODO:
> * How to run your program?
> * What is the meaning of the executing command (both Mininet and Ryu controller)?
> * Show the screenshot of using iPerf command in Mininet (both `SimpleController.py` and `controller.py`)

#### Run program
* open 2 terminals( t1 & t2 )
* first run SimpleTopo.py / topo.py in t1 by "mn --custom SimpleTopo.py / topo.py --topo topo -link tc --controller remote"
* then run SimpleController.py / controller.py in t2 by "ryu-manager SimpleController.py / controller.py --observe-links" 
* use iperf command in t1 and get result
* leave t1 first, then leave t2

#### Meaning
##### Mininet
* --custom  .py: read custom classes or params from .py file(s)
* --topo linear/minimal/reversed/single/torus/tree: linear=LinearTopo, torus=TorusTopo, tree=TreeTopo,
single=SingleSwitchTopo, reversed=SingleSwitchReversedTopo,  minimal=MinimalTopo  
* -link default/ovs/tc: default=Link ovs=OVSLink tc=TCLink  
* --controller default/none/nox/ovsc/ref/remote/ryu: ovsc=OVSController, none=NullController,
                        remote=RemoteController, default=DefaultController, nox=NOX, ryu=Ryu, ref=Controller  
* -c : clean up

##### Ryu controller
* ryu-manager: ryu-manager is the executable for Ryu applications. ryu-manager loads Ryu applications and run it.
* --observe-links: observe link discovery events.  
  
* Screenshot of SimpleController
![](https://i.imgur.com/az9EGHJ.png)
* Screenshot of controller
![](https://i.imgur.com/of44OG2.png)

---
## Description

### Tasks

> TODO:
> * Describe how you finish this work in detail

1. Environment Setup  
* clone initial repository from github and login to container by SSH
* run Mininet with OvS's controller ( to support topos )

2. Example of Ryu SDN  
* open 2 terminals(t1 & t2 )and both login to container
* both change directory to /src/
* first run SimpleTopo.py in t1 by "mn --custom SimpleTopo.py --topo topo -link tc --controller remote"
* then run SimpleController.py in t2 by "ryu-manager SimpleController.py --observe-links"
  
##### leave RYU controller way:
* first leave t1 by "exit"
* then leave t2 by Ctrl-z & "mn -c"
![](https://i.imgur.com/VIaislC.png)

3. Mininet Topology  
* duplicate SimpleTopo.py and change it name to  topo.py by " cp SimpleTopo.py topo.py"
* change s1-s2 & s1-s3 & s2-s3 's bandwidth, delay, loss rate as follow
![](https://i.imgur.com/gmYIMtv.png)
* go to task 5

4. Ryu Controller  
* duplicate SimpleController.py and change it name to  controller.py by " cp SimpleController.py controller.py"
* modify switch_features_handler(self, ev) part as follow rules
![](https://i.imgur.com/uUgB2rR.png)
* go to task 5

5. Measurement  
for task 3(SimpleController):
* first run topo.py in t1 by "mn --custom topo.py --topo topo -link tc --controller remote"
* then run SimpleController.py in t2 by "ryu-manager SimpleController.py --observe-links"
* use iperf commands in t1:  
  h1 iperf -s -u -i 1 -p 5566 > ./out/result1 &  
  h2 iperf -c 10.0.0.1 -u -i 1 -p 5566  
* then get result 1 (leave RYU controller)
  
for task 4(controller):
* first run topo.py in t1 by "mn --custom topo.py --topo topo -link tc --controller remote"
* then run controller.py in t2 by "ryu-manager controller.py --observe-links"
* use iperf commands in t1:  
  h1 iperf -s -u -i 1 -p 5566 > ./out/result2 &  
  h2 iperf -c 10.0.0.1 -u -i 1 -p 5566  
* then get result 2 (leave RYU controller)  

### Discussion

> TODO:
> * Answer the following questions

1. Describe the difference between packet-in and packet-out in detail.  
   packet-in: 由switch發出，請求controller  
   packet-out: 由controller發出，告知switch將packet導致對應主機  
     
2. What is “table-miss” in SDN?  
   假如一個封包在Flow Table裡?有發現能夠匹配成功的Flow Entry，這就是一次"Table Miss"。  
    至於發生Table Miss後的動作取決于此Flow Table的配置：  
    1）直接丟棄，2）繼續轉發給後面的Flow Table，3）封裝成 packet-in 消息送給Remote Controller。
      
3. Why is "`(app_manager.RyuApp)`" adding after the declaration of class in `controller.py`?  
   讓class繼承 app_manager.RyuApp。 app_manager.RyuApp是Ryu應用程式的基礎類別  
     
4. Explain the following code in `controller.py`.
    ```python
    @set_ev_cls(ofp_event.EventOFPPacketIn, MAIN_DISPATCHER)
    ```  
@set_ev_cls(ev_cls, dispatchers=none) 是一個用於將方法註冊成 Ryu 事件處理器的一個修飾器，被修飾的方法將會成為一個事件處理器。  
ev_cls 表示想要被該 Ryu 應用程式接收的事件類別。  
dispatchers 則表示了該事件處理器將會在哪些談判階段（negotiation phases） 去接收此一類型的事件  
MAIN_DISPATCHER: 接收 Switch-features 訊息以及 傳送 set-config 訊息  
  
5. What is the meaning of “datapath” in `controller.py`?  
   the switch in the topology using OpenFlow 
   
6. Why need to set "`ip_proto=17`" in the flow entry?  
  定義該封包資料區使用的協定，17 代表UDP
   
7. Compare the differences between the iPerf results of `SimpleController.py` and `controller.py` in detail.  
  Simple: 10sec 1.21MB 1.01 Mb/s  0.005ms  32/893(3.6%)  
  control: 10sec 1.23MB 1.03Mb/s 0.006ms  18/893(2%)  
 
8. Which forwarding rule is better? Why?  
lost:  simplecontroller > controller
>> controller is better.

---
## References

> TODO: 
> * Please add your references in the following

* **Ryu SDN**
    * [Ryubook Documentation](https://osrg.github.io/ryu-book/en/html/)
    * [Ryubook [PDF]](https://osrg.github.io/ryu-book/en/Ryubook.pdf)
    * [Ryu 4.30 Documentation](https://github.com/mininet/mininet/wiki/Introduction-to-Mininet)
    * [Ryu Controller Tutorial](http://sdnhub.org/tutorials/ryu/)
    * [OpenFlow 1.3 Switch Specification](https://www.opennetworking.org/wp-content/uploads/2014/10/openflow-spec-v1.3.0.pdf)
    * [Ryubook 說明文件](https://osrg.github.io/ryu-book/zh_tw/html/)
    * [GitHub - Ryu Controller 教學專案](https://github.com/OSE-Lab/Learning-SDN/blob/master/Controller/Ryu/README.md)
    * [Ryu SDN 指南 – Pengfei Ni](https://feisky.gitbooks.io/sdn/sdn/ryu.html)
    * [OpenFlow 通訊協定](https://osrg.github.io/ryu-book/zh_tw/html/openflow_protocol.html)
* **Python**
    * [Python 2.7.15 Standard Library](https://docs.python.org/2/library/index.html)
    * [Python Tutorial - Tutorialspoint](https://www.tutorialspoint.com/python/)
* **Others**
    * [Cheat Sheet of Markdown Syntax](https://www.markdownguide.org/cheat-sheet)
    * [Vim Tutorial – Tutorialspoint](https://www.tutorialspoint.com/vim/index.htm)
    * [鳥哥的 Linux 私房菜 – 第九章、vim 程式編輯器](http://linux.vbird.org/linux_basic/0310vi.php)

---
## Contributors

> TODO:
> * Please replace "`YOUR_NAME`" and "`YOUR_GITHUB_LINK`" into yours

* [吳宜珈](https://github.com/oao519p)
* [David Lu](https://github.com/yungshenglu)

---
## License

GNU GENERAL PUBLIC LICENSE Version 3
