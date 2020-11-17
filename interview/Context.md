# React 的上下文-Context

## 导语

- redux 解决的核心问题是:
  - 父子兄弟等组件件传值很麻烦的问题;
- 于是有了一个"通讯班"--redux:
  - 这个通讯班可以帮我们把:
    - 组件之间的状态整合到一起;
      - 然后修改也统一修改;

---

- 但是，修改完成以后，把修改的东西再通知下去我们又会觉得是一个麻烦;
  - 我们也希望能有这样一个通讯班, 帮我们把命令传达下去;
- 为了解决这个问题:
  - react-redux 义无反顾的出现了;
- 这样相对来说就比较完美了;
- 那今天我们就会想把整个工作流都自己来简单实现了:
  - 也便有了接下来的故事;
    - redux、react-redux 兄弟同心，齐力传值;

## redux三板斧

- 