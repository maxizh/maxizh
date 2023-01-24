整个LTE TB_buffer的架构设计都是基于data flow的channel interleaver的行进列出；
如果仅仅是行进行出或者列进列出，那只用fifo来简单握手brp和srp即可，此时还需要注意的点:

  * 如果brp写fifo快，srp读fifo慢，需要fifo full信号来反压brp使不写，也就是brp_wr = ˜fifo_full
  * 如果brp慢srp快，需要fifo empty信号来反压srp使不读，也就是srp_rd = ˜fifo_empty

现在回到legacy design，brp需要整包subframe数据都encode完，srp才能启动。这是保险的做法。

事实上，只要前两行encode完，srp就可以开始每cycle读走2个modulated symbol了。

TODO:

1. 评估brp和srp极限处理速率
2. 梳理ping/pong架构下连续subframe的处理流程

---
今天梳理了sw调度hw，即下tigger的时间点的流程。
* LTE固定20MBW,hw跑208M clk,brp处理时间大概420us,实际hw跑225M,所以8%margin足够
* sw会在420us后下空口td trigger.
* fd trigger在td trigger前1个symbol(70us)下.

另外trace了brp两种trigger的工作模式
* 一种是sw trigger，i.e. sw_txenc_start, 寄存器是w1c类型，所以下完以后被hw清掉，然后brp的fsm开始动
* 另一种是timer trigger，sw先配enable电平高，等ttr那边数时间到了以后送来pulse，enable信号被拉低，brp fsm开始动
