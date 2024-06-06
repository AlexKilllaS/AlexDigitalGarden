## 1. Training Data Format
rasa使用yaml作为训练数据（including NLU data, stories and rules）和domain数据的格式。
```yaml
version: "3.1"

nlu:
- intent: greet
	examples: |
	- Hey
	- Hi
	- hey there [Sara](name)

- intent: faq/language
	examples: |
	- What language do you speak?
	- Do you only handle english?

stories:
- story: greet and faq
	steps:
	- intent: greet
	- action: utter_greet
	- intent: faq
	- action: utter_faq

rules:
- rule: Greet user
	steps:
	- intent: greet
	- action: utter_greet
```

### 1.1 NLU
#### Data Format
NLU训练数据根据intent分类，其中会包括用户的utterance例子。这些例子中，可以包含entities。除了intent，nlu字段下还可以增加synonyms(同义词)，regex(正则表达式)，lookup tables(查询表)。
1. **Entities（实体）**
entity实体是从用户utterance中，能提取的特定语片信息。例如：电话号码、人名、地点等
例子：
```yaml
nlu:
- intent: check_balance
	examples: |
	- how much do I have on my [savings](account) account
	- how much money is in my [checking]{"entity": "account"} account
```
entity语法：
**`[<entity-text>]{"entity": "<entity name>", "role": "<role name>", "group": "<group name>", "value": "<entity synonym>"}`**

2. **synonyms(同义词)，regex(正则表达式)，lookup tables(查询表)**
比较好理解，具体见例子：
```yaml
nlu:
- synonym: credit
	examples: |
	- credit card account

- regex: account_number
	examples: |
	- \d{10,12}

- credit account
- lookup: banks
	examples: |
	- JPMC
	- Bank of America
```
#### [Training Data](https://rasa.com/docs/rasa/nlu-training-data)
NLU (Natural Language Understanding)的目的，是从一句话中，分析出用户的intent和entities。我们可以通过增加规则（regex和lookup tables）来优化模型提取效率。

### 1.2 Conversation Training Data
#### **Stories**
A story is a representation of a conversation between a user and an AI assistant, converted into a specific format where user inputs are expressed as intents (and entities when necessary), while the assistant's responses and actions are expressed as action names.
```yaml
stories:
- story: collect restaurant booking info # name of the story - just for debugging
	steps:
	- intent: greet # user message with no entities
	- action: utter_ask_howcanhelp
	- intent: inform # user message with entities
		entities:
		- location: "rome"
		- price: "cheap"
	- action: utter_on_it # action that the bot should execute
	- action: utter_ask_cuisine
	- intent: inform
		entities:
		- cuisine: "spanish"
	- action: utter_ask_num_people
```
1. [User Messages](https://rasa.com/docs/rasa/training-data-format/#user-messages "Direct link to heading")
user messages用intent和可选的entities key来描述，

```yaml
stories:
- story: user message structure
steps:
	- intent: intent_name # Required
	entities: # Optional
	- entity_name: entity_value
	- action: action_name
```
2.  [Actions](https://rasa.com/docs/rasa/training-data-format#actions)
actions有两种：一种是Responses，用于返回具体的用户回复，以`utter_`开头。
另一种是Custom actions，可用于运行指定代码并返回任意数量的信息。以`action_`开头。
3. [Forms](https://rasa.com/docs/rasa/training-data-format#forms)
4. [Slots](https://rasa.com/docs/rasa/training-data-format#slots "Direct link to heading")
放在`slot_was_set`字段下，以slot name和slot's value的形式存储。相当于一个对话上下文中的存储变量，用来跟踪对话中的关键信息。当一个entity被识别后，它可以被储存到一个slot中，这样对话系统就可以在必要时取用这个信息。
可以被[`action_extract_slots`](https://rasa.com/docs/rasa/default-actions#action_extract_slots)和custom actions设置。在story中，可以被用于判断是否执行后续action。
5. [Checkpoints](https://rasa.com/docs/rasa/training-data-format#checkpoints)
放在story的起始和结尾位置，用于连接不同的story。如果在结尾，这个story可以和以相同checkpoint起始的story连接起来。checkpoint放在开头，还可以用于slots判断：
```yaml
stories:
- story: story_with_a_conditional_checkpoint
steps:
- checkpoint: greet_checkpoint
	# This checkpoint should only apply if slots are set to the specified value
	slot_was_set:
	- context_scenario: holiday
	- holiday_name: thanksgiving
- intent: greet
- action: utter_greet_thanksgiving
```
6. [OR statement](https://rasa.com/docs/rasa/training-data-format/#or-statement)
```yaml
stories:
	- story:
	steps:
		- intent: greet
		- action: utter_greet
		- intent: tell_name
		- or:
			- slot_was_set:
				- name: joe
			- slot_was_set:
				- name: bob
```
#### **Rules**
rule和story很相似，除steps之外，rule还存在`conversation_started` 和 `conditions` keys.
```yaml
rules:
- rule: Only say `hey` when the user provided a name
	condition:
	- slot_was_set:
		- user_provided_name: true
	steps:
	- intent: greet
	- action: utter_greet
```



#### rules和stories的区别
1. Story (故事)：Story是一种描述对话流程的方式，它通过一系列的用户意图和机器人响应来展示对话的可能路径。Story通常用于训练对话管理模型（Dialogue Management Model），使其能够根据历史对话上下文预测下一步的行动。Story不是固定的对话脚本，而是提供了多种可能的对话路径，以便模型能够灵活地处理不同的对话情境。 
2. Rule (规则)：Rule是一种更为严格和确定的对话定义方式，它用于定义对话中的固定逻辑，即在特定情况下总是应该发生的对话步骤。规则不需要通过机器学习来预测，而是直接告诉对话管理器在特定条件下执行哪些行动。例如，当用户说“再见”时，总是回复“再见，有需要再联系我！”这样的对话逻辑就可以通过规则来实现。 总结来说，story提供了灵活的对话路径，适用于复杂和多变的对话场景，而rule定义了确定的对话逻辑，适用于那些应该始终按照特定方式处理的对话步骤。在实际应用中，Rasa通常会结合使用这两种方式来构建更加自然和高效的对话系统。
#### Domain
