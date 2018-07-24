# ERC20标准API:Approve-TransferFrom机制
## approve 授权spender账户地址从本人账户地址转tokens个币
方法签名如下：  
function approve(address spender, uint tokens) public returns (bool success);
## transferFrom从from账户地址转tokens个币到to账户地址，并修改授权余额
方法签名如下：  
function transferFrom(address from, address to, uint tokens) public returns (bool success);  
这两个方法按顺序组合使用，完成授权、转账的机制被广泛使用。该机制是有漏洞的，容易受到攻击。

# 一种可行的攻击方法
- A调用Approve方法，授权B从A账户转账N(N>0)个token.A第一次调用approve,参数为N.
- 某些原因过段时间A想改变N为M(M>0).A第二次调用approve,参数为M.
- 在A第二次调用approve的之前，B调用transferFrom，从A账户转走N个token.
- 在A第二次调用approve的之后，B调用transferFrom，从A账户转走M个token.  
这样B就从A的账户转走M+N个token,显然是违背A的意愿，B攻击成功。

# 原因分析
approve 方法直接覆盖授权数量，忽略当前的授权余额是造成这个漏洞的原因。

# 应对办法
## 方法一 建议修改ERC20 API ：
如果能在修改授权数量前检查授权余额是否发生变化就可以避免。建议修改ERC20 API Approve为:  
function approve(address spender, uint256 currentValue, uint256 value) returns (bool success);

## 方法二 变直接设定授权量为增减授权余额
新增两个方法用来增减授权余额，不使用approve方法。  
function decreaseApprove(address spender, uint tokens) public returns (bool success);  
function increaseApprove(address spender, uint tokens) public returns (bool success);

## 参考
https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit#
