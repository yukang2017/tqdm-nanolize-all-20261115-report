# 本次运行的问题与现象

## 1. 目录已有干净克隆（环境现象）
- 阶段：准备；job：尚未创建。
- 使用已存在的官方克隆，提交9cf5a12；本次证据放在tqdm-2，未覆盖旧记录。

## 2. Graphify未安装（环境现象）
- 阶段：发现；job：尚未创建。
- command -v graphify 无结果。按技能用静态源码证据构图，范围仍为全部功能。

## 3. 避开已知失效的pip附加源（环境配置）
- 阶段：测试环境；job：尚未创建。
- 旧运行记录显示附加源DNS失败。本次prepare_test_environment建立空环境后，以PIP_CONFIG_FILE=/dev/null、PIP_INDEX_URL=https://pypi.org/simple、PIP_EXTRA_INDEX_URL空安装已审查依赖。日志notes/install.log；不改变全局配置。

## 4. 严格JSON Schema兼容（nanolize bug，使用已有修复适配器）
- 阶段：运行器准备；job：尚未创建。
- 旧运行确认设计schema可选content不在required，Codex拒绝HTTP400 invalid_json_schema。
- 本次使用notes/codex_schema_adapter.py，将可选字段转为nullable严格字段，返回后还原省略语义。不改测试断言或插件代码；真实试调用结果另记。

- Schema适配器真实试调用通过：session/answer/usage均为true；14035 tokens，4.658秒。

## 5. 测试启动早于依赖安装完成（会话调度错误，非nanolize bug）
- 阶段：原版预检；job：尚未创建。
- 错误：No module named pytest。
- 原因：并行安装尚未完成时启动测试。
- 处理：保留notes/original-pytest.log；等待pip退出0，重新完整运行，日志notes/original-pytest-complete.log。

## 6. 外部契约转换剔除内部耦合测试（预期机制）
- 阶段：契约转换；job：尚未创建。
- 5组：pytest-tqdm、pytest-synchronisation、pytest-main、pytest-concurrent、pytest-contrib_logging。
- 保留15组，未检查组为0、carry文件0；补充所有叶节点的外部语义契约，由作业冻结时自动设计行为测试。

## 7. 原版测试的可选环境缺口（环境现象）
- 阶段：原版预检；job：ac3103d50e924f80bcee05809c8fe140。
- 181 passed、5 skipped，13.50秒；3项concurrent.interpreters/Python3.14、1项itertools.batched、1项Keras。
- 不安装大型条件框架；保留全部功能并如实记录未验证项。

## 8. 使用本机执行器（环境设置）
- 阶段：提交；job：ac3103d50e924f80bcee05809c8fe140。
- 旧记录显示Slurm控制器DNS不可解析，本次明确check_executor.kind=local，避免自动探测失败。未放宽任何检查。

## 9. 上游测试事实覆盖不足（上游测试现象，非nanolize bug）
- 阶段：测试事实审查；job：ac3103d50e924f80bcee05809c8fe140。
- test-facts.json记录9项uncovered_tests：7项perf、2项utils；另4条事实被审查退回。
- tests_utils.py中的assert 1, ...和assert 1.1, ...只验证常量真值，后半部分是未执行的错误消息表达式，未调用被测函数。
- 处理：保留原版测试和审查记录，继续节点有效性验证与自动测试设计；没有修改断言。

## 10. 观察脚本读取了尚未写完的JSON（会话观察错误，非nanolize bug）
- 阶段：测试设计观察；job：ac3103d50e924f80bcee05809c8fe140。
- 错误：JSONDecodeError: Expecting value。
- 原因：result.json可先建立空文件；文件存在不等于调用结束。
- 处理：统计只接受可解析的结果，保留正在写入的文件；没有改动作业。

- 另一次观察脚本把scoped baseline包装记录误作底层baseline读取results，产生KeyError；改为先检查schema再读取。不影响作业或测试。

## 11. 第一轮补测不具区分能力（测试设计现象，非已确认nanolize bug）
- 阶段：自动测试设计；job：ac3103d50e924f80bcee05809c8fe140。
- 4个文件被拒绝：examples 1个、terminal 3个；原因均为no_behavioral_knockout_failure，即节点功能失效后测试仍通过。
- 处理：作业自动启动这两个节点的第二轮设计，维持有效性判据。最终结果见test-design.json。
- 后续事实审查又退回Slack设计文件：测试断言update()和update(0)返回None，但随测试提交的事实未声明该行为（reviewer_veto）。作业启动round-3/slack补齐，属于测试与事实不一致的设计现象。

## 12. 失效注入破坏导入与保留但无效覆盖的检查（验证现象/局限，未确认为nanolize bug）
- 阶段：补测验收；job：ac3103d50e924f80bcee05809c8fe140。
- Discord、GUI、Telegram、Tk出现knockout_breaks_import，不能以导入失败证明行为覆盖。4项被记录在runner_failures字段，但原因不是模型调用失败。
- examples的1个和terminal的2个测试通过原版及审查，但失效注入后仍通过；作为pinned checks保留，两节点仍标记缺少有效测试。
- 处理：遵循默认非严格策略继续；不将pinned等同有效覆盖，不改变功能范围或验收断言。20个有效设计文件覆盖11叶节点，10个提案被拒绝。
- 10个被拒绝提案还包括release的1个动态属性访问提案，原因outside_external_contract；与有效性不足4个、导入破坏4个、Slack事实不一致1个合计10个。此项属于契约边界拦截现象。


## 13. 发布辅助工具的契约细节不足（契约映射现象，非已确认nanolize bug）
- 阶段：第二次独立实现调用；job：ac3103d50e924f80bcee05809c8fe140。
- 作者日志请求补充release脚本路径及doc2rst、README、snapcraft.yaml的输入输出格式。
- 原因判断：初始图契约补充了发布工具的公开语义，但部分细节没有被现有检查和事实明确固定。
- 处理：保持冻结契约，交由后台控制器和既有事实继续处理；观察者没有向作者传递原版代码或测试。最终通过结果仅对冻结检查负责。

## 14. 首个候选的功能回归和检查改写源码（候选实现问题，非nanolize bug）
- 阶段：候选1验收；job：ac3103d50e924f80bcee05809c8fe140。
- 38组检查36通过、2失败：CLI信息输出与资源安装、release补全生成；独立安装通过。
- 验证器同时报告checks_modified_inputs，路径tqdm/completion.sh，阻止接受在检查中改变源码的候选。
- 处理：交给后台控制器使用同一冻结基线自动修复；未修改测试、基线或关闭保护。后续结果续记。
- 作者修复日志定位：布尔CLI选项帮助含多余值占位；安装资源应逐字节复制；补全生成应写入被调用脚本所在副本，而非导入helper所在目录。修复后等待冻结检查确认。

- 候选2全量复测：37/38组通过，剩1组失败；checks_modified_inputs已消失，安装仍通过。后台继续第二轮修复。

## 15. 测试事实丢失精确资源格式（已确认的nanolize契约/审查链缺陷，需要改进）
- 阶段：CLI资源安装修复；job：ac3103d50e924f80bcee05809c8fe140。
- 证据：冻结知识fact.717771ae38f80e2f.9将man页描述为TH指令命名TQDM和section 1、SYNOPSIS中options斜体；对应冻结测试却逐字检查引号、尾随空格和troff字体复位转义。
- 遗漏：事实没有保留这些决定测试通过与否的字面格式；两个不同的TH排版都满足事实的语义概述，但严格测试只接受其中一种。候选2、3同样剩2条安装断言失败；作者日志明确表示反馈未给出更窄诊断，修复变成猜测。
- 分类：确认存在事实表达与验收条件的信息不一致；需要nanolize在测试事实生成/审查和反馈链中保留并核验被逐字断言的公共输出，不应仅用语义概述覆盖测试。这不是原版库bug，也不等于证明模型生成器的某一行Python有确定性错误。
- 处理：保留证据，未改测试或冻结知识，未向独立作者传递原版测试内容；让现有自动修复继续。建议修复事实保真与审查覆盖，而非放松验收。
- 候选1至5的用例失败数为5、2、2、2、2；后续修复未消除同一组man资源安装失败。控制器停止修复，恢复保存的候选为candidate-6，并重新执行全量验证后按带失败结果交付。没有降低验收要求。

## 16. 自动公开发布首次失败（发布现象，原因未确认）
- 阶段：公开发布；job：ac3103d50e924f80bcee05809c8fe140。
- 错误：GitHub report request failed; check gh login and repository permissions。
- 检查发现登录与读取权限正常，公开仓库已建立。使用publish_report_site对同一安全站点重试，无需改凭据、权限或插件代码，返回pending部署状态。
- 分类：尚无证据证明权限不足或nanolize确定性缺陷；可能是外部API或新建资源时序问题，不能下定论。后续已通过匿名访问与报告哈希验证，报告状态为published。

## 17. GitHub连接器无新仓库写权限（集成权限现象，非nanolize bug）
- 阶段：中文汇总页发布；job：ac3103d50e924f80bcee05809c8fe140。
- 错误：GitHub API error 403: Resource not accessible by integration。
- 原因：连接器不能向新公开报告仓库写文件；与本机gh登录的权限不同。
- 处理：使用此前已成功发布原生报告的本机gh/Git凭据，在同一gh-pages分支添加汇总与问题文件，不改变仓库可见性或账户权限。

## 18. HTTPS已可用但发布状态持续pending（确认的nanolize发布状态兼容缺陷）
- 阶段：匿名发布验证；job：ac3103d50e924f80bcee05809c8fe140。
- 证据：GitHub Pages已built，API的html_url为http://自定义域名路径；nanolize的_verify_url对非https直接返回false，状态持续pending且不提供公开URL。实际HTTPS匿名GET为200，报告SHA256与本地完全一致。
- 原因：发布器建立站点后未处理“HTTP元数据地址但HTTPS已可用”的情况，也未自动完成本次新报告站点的HTTPS配置。
- 设置规避：只对本次新建报告仓库Pages设置https_enforced=true，API地址随即改为HTTPS，状态工具开始提供正确expected_url；部署完成后状态工具已返回published，anonymous_access_verified=true；HTTPS报告哈希与本地完全一致。
- 建议nanolize在发布流程中处理HTTPS配置或安全探测可用的HTTPS规范地址，不应把已经可验证的报告永久留在pending。没有改插件代码或全局自定义域名。

## 19. 浏览器验证环境缺少动态库（环境现象，非nanolize bug）
- 阶段：公开页面视觉验证；job：ac3103d50e924f80bcee05809c8fe140。
- Chromium首次启动失败：libnspr4.so、libnss3.so、libnssutil3.so缺失；系统APT包索引也未准备，初次download无法定位包。
- 处理：在/tmp下建立浏览器专用包索引、缓存和库目录，下载解包依赖，启动时仅为该进程设置库路径；不改系统安装或全局配置。另在临时目录解包中文字体。最终桌面1440×1000与手机390×844的原生功能图、中文汇总均通过视觉及交互检查，0个JavaScript错误、无页面横向溢出。

- 浏览器观察脚本曾按可见文字定位节点，但实际ARIA名称带Inspect前缀，导致一次定位超时；按正确ARIA标签重跑后通过。这属于观察脚本问题，非页面交互故障。
