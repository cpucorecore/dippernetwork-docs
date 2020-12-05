# 链上治理

## 概念

链上治理过程可以分为以下几个步骤：

- **提议提交:**
  
  将提议发送至链上，同时抵押一定数量的 DIP。

- **投票:**

  一旦抵押数量达到某个特定值（`最小抵押`），提议将被确认，对该提议的投票也将开启。之后，质押的 DIP 持有者可以发送 `TxGovVote` 交易对此提议进行投票。

- 当提议涉及软件升级时，流程请参考 [`Upgrade`](upgrade.md)

### 提议提交

#### 提交提议的权利

无论质押与否，任何 DIP 持有者均可通过发送 `TxGovProposal` 交易来提交提议。一旦被提交，将生成一个唯一的 `proposalID` 来标识提议。

#### 提议类型

在当前版本中，支持两种类型的提议：

- `纯文本提议`
  
  所有不涉及源代码修改的提议都被归类为这种类型。例如，一个民意调查将使用一个 `纯文本提议` 类型的提议。

- `软件升级提议`

  请参考 [`Upgrade`](upgrade.md)。

其他模块可以通过实现自己的提议类型和处理器来扩展治理模块。这些类型通过治理模块注册和处理（例如 `参数改变提议`）。当一个提议通过时，治理模块将执行相关模块的提议处理器。自定义处理器可以完成任意的状态改变。

### 抵押

为防止垃圾信息，提议必须用 `最小抵押` 参数中定义的币种进行抵押。只有当一个提议的抵押数量满足 `最小抵押` 时，投票期才能开始。

当提议被提交时，必须进行抵押；抵押数量必须大于0，但可以小于 `最小抵押`。提交者不需要自己支付整个押金。如果一个提议的押金不满足 `最小抵押` 时，其他的代币持有者可以通过发送 `Deposit` 交易增加该提议的押金。押金被托管在治理模块的 `模块账户`中，直到该提议终止（通过或拒绝）。

一旦押金达到 `最小抵押`，提议将进入投票期。如果押金在 `最大抵押期` 前没有达到 `最小抵押`，提议将关闭并且任何人都不能再为此提议增加抵押。

#### 押金退还与销毁

当一个提议终止后，根据最终的合计结果，该提议的押金将被退还或者销毁：

- 如果提议被批准或者被拒绝（但没有被驳回），押金将自动退还到相应的抵押地址（从治理模块的 `模块账户` 转移）。

- 当提议被绝大多数驳回时，押金将从治理模块的 `模块账户` 销毁。

### 投票

#### 参与者

_参与者_ 是对提议有投票权利的用户。在 Dipper Network Hub 上，参与者是质押的 DIP 持有者。未质押的 DIP 持有者和其他用户无权参与治理。但是他们能提交提议及抵押。

注意，对于一个特定的验证人，如果一些参与者满足以下条件，则这些参与者可以被禁止为一个提议投票：

- 在提议进入投票期后，参与者对该验证人质押或解质押 DIP
  
- 在提议进入投票期后，参与者成为验证人

但是这不阻止参与者用质押到其他验证人的 DIP 进行投票。例如，在一个提议进入投票期之前，参与者质押一些 DIP 到验证人 A；在该提议进入投票期之后，质押另一些 DIP 到验证人 B；则仅仅在验证人 B 下的投票将被禁止。

#### 投票期

一旦一个提议达到 `最小抵押`，该提议立即进入 `投票期`。我们把 `投票期` 定义为投票开启时刻和关闭时刻的间隔。为阻止重复投票，`投票期` 应该总是小于 `解质押期`。`投票期` 的初始值为 2 周。

#### 选项集

The option set of a proposal refers to the set of choices a participant can
choose from when casting its vote.

一个提议的选项集指的是，当一个参与者进行投票时可以从中选择的选项集合。

初始选项集包括下列的选项：

- `Yes`
- `No`
- `NoWithVeto`
- `Abstain`

The initial option set includes the following options:

- `Yes`：赞成
- `No`：反对
- `NoWithVeto`：否决
- `Abstain`：弃权

`NoWithVeto` 记为 `No`，但同时也增加一个 `否决` 投票。`Abstain` 选项允许投票者表示他们不打算投票赞成或反对提议，但接受投票结果。

_注意: 在 UI 端，我们可以为一些紧急的提议增加一个 `Not Urgent`（`不紧急`）选项，该选项表示一个 `NoWithVeto` 投票。_

#### 法定人数

Quorum is defined as the minimum percentage of voting power that needs to be
casted on a proposal for the result to be valid.

法定人数被定义为，为使一个提议的投票结果有效，所需要进行投票的投票权的最小百分比。

#### 阈值

Threshold is defined as the minimum proportion of `Yes` votes (excluding `Abstain` votes) for the proposal to be accepted.

阈值被定义为，为使提议被接受而所需要的 `赞成` 票的最小比例。

阈值初始被设置为 50%，如果超过 1/3 的投票（排除 `弃权` 投票）是 `否决` 票，有一定的否决概率。这意味着在投票期结束时，如果 `赞成` 投票（排除 `弃权` 投票）的比例大于 50%，并且 `否决` 投票（排除 `弃权` 投票）的比例小于 1/3，则提议将被接受。

Proposals can be accepted before the end of the voting period if they meet a special condition. Namely, if the ratio of `Yes` votes to `InitTotalVotingPower`exceeds 2:3, the proposal will be immediately accepted, even if the `Voting period` is not finished. `InitTotalVotingPower` is the total voting power of all bonded Iris holders at the moment when the vote opens.
This condition exists so that the network can react quickly in case of urgency.

在投票期结束之前，如果满足一个特殊条件，提议可以被接受。即，如果 `赞成` 投票与 `InitTotalVotingPower`（`初始总投票权`）的比例超过 2:3，提议将立即被接受，即使 `投票期` 还没结束。`初始总投票权` 是投票开启时所有质押的 Iris 持有人的总投票权。这个条件存在以至于在紧急情况下网络可以快速做出反应。

#### 继承

如果一个委托人没有投票，它将继承它的验证人的投票。

- 如果委托人在它的验证人之前进行投票，它将不会继承验证人的投票。

- 如果委托人在它的验证人之后进行投票，它将用自己的投票覆盖验证人的投票。如果提议紧迫，投票可能在委托人有机会作出响应并且覆盖相应验证人的投票之前结束。这不是一个问题，因为在投票期结束前，提议需要多于 2/3 的投票权才能通过。如果多于 2/3 的验证人合谋，他们也能审查委托人的投票。

#### 对验证人不投票的惩罚

当前，验证人不会因不投票而受到惩罚。

#### 治理地址

在之后的版本中，我们可以引入许可地址对特定模块的交易进行签名。作为 MVP，`治理地址` 将是在账户创建时产生的主验证人地址。这个地址对应一个与为共识消息签名的 Tendermint 私钥不同的私钥。因此验证人不必用敏感的 Tendermint 私钥签署治理交易。

### 软件升级

关于软件升级的治理过程，参见 [`Upgrade`](upgrade.md)。

## 使用场景

### 参数改变

模块的参数可以通过一个参数改变提议进行修改。

### 社区池基金消费

社区池基金可以通过治理过程进行消费。

```bash
# 提交社区池基金消费提议
echo '{
  "title": "Community Pool Spend",
  "description": "Developer rewards",
  "recipient": "dip1fcjdh96hnnc79fzwytv02svkkqqk8gfxmsuv4u",
  "amount": "10000pdip",
  "deposit": "1000pdip"
}' > proposal.json

iris tx gov submit-proposal community-pool-spend proposal.json --from=<key-name> --fees=0.3iris --chain-id=irishub
```

### 软件升级

软件升级的用法请参考 [`Upgrade`](upgrade.md)