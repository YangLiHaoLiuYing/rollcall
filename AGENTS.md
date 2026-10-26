# AGENTS.md — 给落在这个仓库里的 agent

这个仓库是 **协同板（rollcall）** skill：多 agent 心跳看板、看门狗、定向问答（ask/reply/await）
和用户喊话通道。零第三方依赖，只用 Python 标准库。

先用哪一半，取决于你来干什么。

## 你要「用它」——把一块板开起来

```bash
python3 scripts/rollcall.py setup
```

这一条命令在 **`<cwd>/.workbuddy/rollcall`** 建板、认领身份、打出唤醒器命令（旧落点 `.workbuddy/work-log` 已存在时接着用它），
并跑真 `doctor` 自检、告诉你还差哪一步。它是**唯一**带默认落点的命令；别的命令
都要显式 `--dir`（或 `$WORK_LOG_DIR`），免得指到别人的板。

要装机器级常驻（会话睡着时也有人认领用户喊话）加 `--apply` —— 它会写
`~/Library/LaunchAgents`，属于项目路径之外，所以默认不动。

装完之后，一个 agent 的最小日常：

```bash
D="<板目录>"
python3 scripts/rollcall.py --dir "$D" post --agent <你> --text "..." --tag 执行  # 心跳，别超过看板的静默阈值
python3 scripts/rollcall.py --dir "$D" check           # 有卡死/险情/没人认领的喊话 ⇒ 退 1
python3 scripts/rollcall.py --dir "$D" brief           # 只投喂增量：别人的新动态 + 用户喊话
python3 scripts/rollcall.py --dir "$D" read-user --agent <你>   # 读用户喊话（会推进你的游标）
python3 scripts/rollcall.py --dir "$D" ack-user --agent <你> --id N --text "我打算做什么"  # 认领
```

别把命令塞进 `WL="python3 … --dir …"` 这种带空格的变量再展开：zsh 不对 `$WL` 做词分割。

唤醒器（`waker`）必须在**你要被叫醒的那个会话**里起 —— 别的进程替你起的哨兵
叫不醒你。`setup` 只把命令打出来，不替你起。

- 装到宿主 skills 目录时，**顶层目录名必须叫 `rollcall`**（宿主按目录名注册 skill，
  不看 `SKILL.md` 里的 `name` —— 目录叫别的名字，宿主里就显示别的名字）。
  交付包 `rollcall-v0.8.0.tar.gz` 解压出来顶层就叫 `rollcall/`。
- 全部命令：[`references/cli.md`](references/cli.md)
- 30 秒上手、协议与纪律：[`SKILL.md`](SKILL.md)
- 用户喊话「没人认领」会进办事轴告警；哨兵的自动回执**不算**认领。

## 你要「改它」——在这个仓库提交

```bash
bash scripts/selftest.sh                  # 只碰临时目录；全绿才算完
bash -n scripts/selftest.sh               # 改了测试脚本必跑
python3 scripts/wl_doc_cmd_check.py scripts/rollcall.py \
    README.md SKILL.md references/usage.md references/protocol.md   # 文档里的命令/旗标对账
bash examples/demo.sh > /tmp/demo_out.txt                # 演示输出必须是真的
python3 scripts/wl_demo_output_check.py . /tmp/demo_out.txt   # README 节选/措辞逐字对账
```

- 改动要配回归断言：没有断言的功能等于没有功能（[`CONTRIBUTING.md`](CONTRIBUTING.md) 有完整清单）。
- 演示输出**不许换皮**：README 的节选必须逐字出自 `examples/demo-output.txt`，
  险情/熔断那几行必须与当场重跑 `examples/demo.sh` 的同一行逐字相同 ——
  `scripts/wl_demo_output_check.py` 会当场抓这两种造假。
- 断言总数只在 `scripts/selftest.sh` 头部声明一次（`ASSERT_TOTAL`）；文档里的数字由
  套件自己的对账组检查，不要手抄第二份。
- 发布验证要在**同一份最终内容**上跑完多个解释器（3.12 / 3.13 / 3.9 / 默认），
  中途不要改被测试的树。
- 用 `--text-file` 传长文本（尤其长中文）：`--text` 走 argv，反引号会被 shell 静默吃掉。
- 不要提交别的线正在改的文件；源代码树改动前先在看板上 `lock`。

## 明确不做的事

- 不猜看板落点（`setup` 除外：它只按 `<cwd>/.workbuddy/rollcall` 建**新**板）。
- 不把密钥、本机私有路径写进看板、文档或提交。
- 不动别人的常驻进程 / 哨兵 / serve；看清 pid 归属再谈重启。
