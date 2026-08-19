# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

质量审核明确拒绝一条温控偏差后，审核记录已保存，但样本仍处于隔离状态，后续对账持续阻塞。请修复拒绝结论的跨实体状态更新。 请只修改必要的生产代码，不得新增、删除或修改测试文件，不得跳过测试或放宽断言。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/coldchain-custody-task-14
- 仓库地址：https://github.com/zhanglei10281852-gif/coldchain-custody-task-14.git
- parent SHA：fe03bd8949dc8e78e478477d39833df793b4d76d

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/coldchain-custody-task-14.git bug-repro
cd bug-repro
git checkout --detach fe03bd8949dc8e78e478477d39833df793b4d76d
go test ./internal/service -run "^TestRejectedReviewDestroysSamples$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestRejectedReviewDestroysSamples$" -count=1
--- FAIL: TestRejectedReviewDestroysSamples (0.49s)
    annotation_behavior_test.go:163: rejected batch = {ID:sample_5370f7f175943d7b65704199 StudyID:study_42142c0fd01eedd3e5f3eb97 OriginSiteID:site_72b72f07635c0e693e4c124e ExternalRef:EXT-1 SpecimenType:plasma VialCount:2 VolumeMilliLit:100 State:quarantined ExpiresAt:2026-08-20 08:00:00 +0000 UTC ShipmentID:ship_f9ee6c48ff12080ddb96b730 QuarantineNote:temperature excursion exc_6b58d714abca2f70914863e1 CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:6}
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/service	0.488s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestRejectedReviewDestroysSamples$" -count=1
--- FAIL: TestRejectedReviewDestroysSamples (1.32s)
    annotation_behavior_test.go:163: rejected batch = {ID:sample_d55f3683bea4ae2f116c6045 StudyID:study_f8cc27cc4c59db77e55ce0d8 OriginSiteID:site_729be234b1e7e93902f21ed9 ExternalRef:EXT-1 SpecimenType:plasma VialCount:2 VolumeMilliLit:100 State:quarantined ExpiresAt:2026-08-20 08:00:00 +0000 UTC ShipmentID:ship_ac58bd09b239fae8da4bee63 QuarantineNote:temperature excursion exc_f79b3477a552a60f0eb5327d CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:6}
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/service	1.515s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

## 通过条件

定向公开行为验证通过，相关包和全量测试通过，go vet 及 linux/amd64 构建通过。 定向命令 go test ./internal/service -run ^TestRejectedReviewDestroysSamples$ -count=1 必须由修复前失败变为修复后通过；相关包与 go test ./... -count=1 全量回归通过，回退 gold 关键修改后定向命令重新失败。
