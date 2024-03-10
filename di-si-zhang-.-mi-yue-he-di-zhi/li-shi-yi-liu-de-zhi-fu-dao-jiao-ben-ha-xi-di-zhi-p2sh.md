# 历史遗留的支付到脚本哈希地址（P2SH）

正如我们在前面的章节中所看到的，接收比特币的人（比如Bob）可以要求支付给他的比特币在其输出脚本中包含某些约束条件。当Bob花费这些比特币时，他将需要使用输入脚本来满足这些约束条件。在“IP地址：比特币的原始地址（P2PK）”中，约束条件简单地是输入脚本需要提供适当的签名。在“用于P2PKH的遗留地址”中，还需要提供适当的公钥。

对于支付给Bob的输出脚本中放置Bob想要的约束条件的支出者（比如Alice），Bob需要将这些约束条件传达给她。这类似于Bob需要将他的公钥传达给她的问题。就像这个问题一样，公钥可以相当大，Bob使用的约束条件也可能非常大——潜在地有数千字节。这不仅是需要传达给Alice的数千字节，而且是她每次想向Bob支付款项时都需要支付交易费用的数千字节。然而，使用哈希函数为大量数据创建小承诺的解决方案在这里也适用。

2012年比特币协议的BIP16升级允许输出脚本承诺一个赎回脚本（redeem script）。当Bob花费他的比特币时，他的输入脚本需要提供一个与承诺匹配的赎回脚本，以及满足赎回脚本所需的任何数据（例如签名）。让我们首先想象一下，Bob想要要求两个签名来花费他的比特币，一个来自他的桌面钱包，另一个来自硬件签名设备。他将这些条件放入一个赎回脚本中：

\<public key 1> OP\_CHECKSIGVERIFY \<public key 2> OP\_CHECKSIG

然后，他使用与P2PKH承诺相同的HASH160机制创建对赎回脚本的承诺，即RIPEMD160（SHA256（脚本））。该承诺被放置到输出脚本中，使用特殊的模板：

OP\_HASH160 OP\_EQUAL

{% hint style="danger" %}
使用支付至脚本哈希（P2SH）时，必须使用特定的P2SH模板，在输出脚本中没有额外的数据或条件。如果输出脚本不是完全是 OP\_HASH160 <20字节> OP\_EQUAL，赎回脚本将不会被使用，而且任何比特币可能要么无法支配，要么可以被任何人支配（意味着任何人都可以取走它们）。
{% endhint %}

当Bob要花费他收到的支付，用于他脚本的承诺时，他会使用一个包含赎回脚本的输入脚本，将其序列化为一个单独的数据元素。他还提供了满足赎回脚本所需的签名，按照它们被操作码消耗的顺序放置：

\<signature2> \<signature1> \<redeem script>

当比特币全节点接收到Bob的交易时，它们将验证序列化的赎回脚本是否会哈希为与承诺相同的值。然后，它们将其替换为堆栈上的反序列化值：

\<signature2> \<signature1> \<pubkey1> OP\_CHECKSIGVERIFY \<pubkey2> OP\_CHECKSIG

脚本被执行，如果通过并且所有其他交易细节都正确，则交易有效。&#x20;

P2SH的地址也使用base58check创建。版本前缀设置为5，这会导致编码地址以3开头。P2SH地址的示例是3F6i6kwkevjR7AsAd4te2YB2zZyASEm1HM。

{% hint style="info" %}
P2SH不一定等同于多重签名交易。P2SH地址通常代表一个多重签名脚本，但也可能代表编码其他类型交易的脚本。
{% endhint %}

\
P2PKH和P2SH是使用base58check编码的仅有两种脚本模板。它们现在被称为传统地址，并且随着时间的推移变得越来越不常见。传统地址已经被bech32地址家族所取代。



> **P2SH碰撞攻击**
>
> 基于哈希函数的所有地址理论上都容易受到攻击者独立找到产生哈希函数输出（承诺）的相同输入的影响。在比特币的情况下，如果攻击者以与原始用户相同的方式找到输入，他们将知道用户的私钥并能够花费该用户的比特币。攻击者独立生成现有承诺的输入的机会与哈希算法的强度成正比。对于像HASH160这样的安全160位算法，这种可能性是1/2^160。这是一种原像攻击。&#x20;
>
> 攻击者还可以尝试生成两个不同的输入（例如，赎回脚本），这些输入会产生相同的承诺。对于完全由单方创建的地址，攻击者生成现有承诺的不同输入的机会也大约是1/2^160，对于HASH160算法而言也是如此。这是二阶原像攻击。&#x20;
>
> 然而，当攻击者能够影响原始输入值时情况就会改变。例如，攻击者参与了多方签名脚本的创建，在这种情况下，他们在了解所有其他参与方的公钥之后不需要提交自己的公钥。在这种情况下，哈希算法的强度降低到其平方根。对于HASH160，概率变为1/2^80。这是一种碰撞攻击。 为了将这些数字放入上下文，截至2023年初，所有比特币矿工每小时执行大约2^80个哈希函数。他们运行与HASH160不同的哈希函数，因此他们现有的硬件无法为其创建碰撞攻击，但比特币网络的存在证明了对HASH160等160位函数的碰撞攻击是切实可行的。比特币矿工已经花费了数十亿美元的特殊硬件，因此创建碰撞攻击不会很便宜，但有些组织预计将获得数十亿美元的比特币到与多方参与的过程相关的地址，这可能会使攻击变得有利可图。 有着成熟的密码协议用于预防碰撞攻击，但一个简单的解决方案，不需要钱包开发人员具有任何特殊的知识，就是简单地使用更强大的哈希函数。比特币的后续升级使这成为可能，新的比特币地址提供了至少128位的碰撞抵抗能力。执行2^128次哈希操作将需要所有当前的比特币矿工大约320亿年。&#x20;
>
> 尽管我们认为没有任何立即威胁到任何人创建新的P2SH地址，但我们建议所有新钱包使用更新类型的地址，以消除地址碰撞攻击的担忧。
