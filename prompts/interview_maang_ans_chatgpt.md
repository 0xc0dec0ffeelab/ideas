# A. 開源挑戰項目（精選 5 個）

## 1) Kubernetes「good first issue / help wanted」

* **平台與連結**：Kubernetes 貢獻指南與標籤說明、議題清單。([Kubernetes Contributors][1], [GitHub][2])
* **技術難度**：中級（閱讀大型專案、寫 E2E/單元測試、修小 bug）
* **主要技術棧**：Go、kubebuilder、ginkgo/gomega、make、kind/minikube
* **參與步驟**

  1. 讀「First Contribution」→ 設定本機環境與簽 CLA。([Kubernetes Contributors][1])
  2. 在 `kubernetes/kubernetes` 以 `good first issue` or `help wanted` 篩選議題。([GitHub][2])
  3. 在議題留言 `/assign` 或回覆「I’d like to take this」並簡述計畫。
  4. 建分支、加測試、提交 PR，依維護者回饋修正。
* **成果展示方法（履歷句型）**

  * 「為 Kubernetes `kubelet` 測試補齊 *X* 個案例，CI 綠燈率 +100%，PR #123456 被合併」
* **量化指標範例**

  * *PR merged = 2、Review rounds ≤ 2、從開工到合併 TTM = 9 天、覆蓋率 +3.1%*

---

## 2) Go 官方專案（`golang/go` & VS Code Go 擴充）

* **平台與連結**：`golang/go` issue 列表與 `HelpWanted` 標籤（含 VS Code Go）。([GitHub][3])
* **技術難度**：中級～高級（讀編譯器/標準庫議題可從工具鏈或文件改善著手）
* **主要技術棧**：Go、`go tool pprof`、`cmd/go`、LSP
* **參與步驟**

  1. 篩選 `help wanted`/`NeedsInvestigation` 的小修（文件、註解、工具錯誤訊息）。([GitHub][4])
  2. 本地重現、加最小可重現測試、送 CL。
* **成果展示（履歷句型）**

  * 「修正 `go vet` 對某語法的誤報，改善開發者體驗；PR 合併入 Go 1.xx」
* **量化指標**

  * *Closed issues = 1、工具錯誤重現步驟縮短 50%、被 release note 點名*

---

## 3) Spring Boot（Java）

* **平台與連結**：`spring-projects/spring-boot` issue 列表，含 bug / enhancement。([GitHub][5])
* **技術難度**：中級（自動配置、測試、文件）
* **主要技術棧**：Java 21、Spring Boot 3/4、JUnit 5、Testcontainers、Micrometer
* **參與步驟**

  1. 找到文件/依賴升級/測試失敗類議題 → 開分支 → 加 `@SpringBootTest` 或切片測試。
  2. 附重現專案（最小化）與 benchmark（JMH）。
* **成果展示**

  * 「提交 `MockRestServiceServerAutoConfiguration` 的可覆蓋性修正，新增 6 個測試，CI 失敗案例 -100%」
* **量化指標**

  * *Unit tests +6、覆蓋率 +2.4%、PR merged 於 3.4.x*

---

## 4) Apache Kafka（JIRA）

* **平台與連結**：Kafka JIRA 專案入口；貢獻經驗分享與常見 bug/角落案例。([Apache Issues][6], [Medium][7], [Confluent][8])
* **技術難度**：中級～高級（client/streams 測試與小修起手）
* **主要技術棧**：Java、Gradle、Kafka Streams、JUnit、Testcontainers-Kafka
* **參與步驟**

  1. 在 JIRA 搜 `newbie`/`starter`/`help wanted`。
  2. 先挑 doc/測試議題：加出界條件測試、重現 race、修 NPE。
* **成果展示**

  * 「為 `FetchRequest` 邊界條件新增測試覆蓋，避免 broker 在 `min.bytes` 下阻塞」
* **量化指標**

  * *Failing IT 減少 3 件、CI wall time -8%、被 release notes 收錄 1 次*

---

## 5) 開源議題聚合器（Good First Issue / Up-for-grabs）

* **平台與連結**：GoodFirstIssue.dev、GitHub Topic、Up-for-grabs。([Good First Issue][9], [GitHub][10])
* **技術難度**：初級～中級（快速找到合適議題）
* **主要技術棧**：依專案而定（優先 Java/Go）
* **參與步驟**

  1. 以語言過濾（Java/Go）→ 用標籤與星等篩小型庫，先從文件/測試開始。
* **成果展示**

  * 「連續 4 週每週 1 PR，橫跨 3 個 repo（1 Java、2 Go）」
* **量化指標**

  * *PR merged 4、首次回覆時間 < 24h、maintainer 正向評語 2 次*

---

# B. GitHub 求職作品集（7 個能打的題目）

---

## 1) 事件驅動電商（Java / Spring）—「Order+Payment Outbox & SAGA」

* **描述**：下單→支付→出貨，用 Outbox + 可靠事件與補償（SAGA）
* **技術棧**：Spring Boot 3、Spring Cloud Stream、Kafka、PostgreSQL、Debezium Outbox、Flyway、Resilience4j、Micrometer + Prometheus + Grafana、Testcontainers、k6
* **功能**

  * 下單（預留庫存）、支付（預授權）、訂單狀態機（Pending→Paid→Shipped / Cancelled）
  * Outbox 表記錄事件，Debezium 將變更推 Kafka，消費者冪等處理
* **性能要求**：*目標* P95 延遲 < 120ms、穩態 QPS 800、失敗率 < 0.5%、庫存一致性 0 lost update
* **核心程式碼（Java：Outbox 寫入 + 針對重試的 `@Transactional` 片段）**

  ```java
  @Service
  public class OrderService {
    private final OrderRepo repo;
    private final OutboxRepo outbox;

    @Transactional
    public OrderId placeOrder(CreateOrder cmd){
      Order o = Order.create(cmd);          // 驗證&預留庫存
      repo.save(o);
      OutboxEvent evt = OutboxEvent.of("OrderCreated", o.getId(), o.toJson());
      outbox.save(evt);                     // 與狀態變更同一交易提交
      return o.getId();
    }
  }
  ```
* **部署**：Docker Compose（Postgres、Kafka、Connect+Debezium、Grafana、Prometheus）。K8s：每個服務 `Deployment`，Kafka 用 Strimzi / MSK。HPA 以 `requests_per_second` 指標擴縮。
* **測試策略**：

  * 單元：狀態機轉移 / 冪等性（同一 `idempotencyKey` 多次呼叫結果一致）
  * 整合：Testcontainers 起 Postgres+Kafka，驗證 outbox→Kafka 事件
  * 壓測：k6 腳本（下方共用）

---

## 2) Go 微服務「高併發短網址平台」

* **技術棧**：Go 1.22、Gin/Fiber、Redis（hot path）、PostgreSQL（冷資料）、Bloom Filter、pprof、OpenTelemetry、Prometheus
* **功能**：建立短連結、跳轉、限流、AB 測試（多個目標 URL）
* **性能要求**：P99 < 25ms、QPS 15k（Redis 命中率 > 95%）
* **核心程式碼（Go：限流 + 熱路徑快取）**

  ```go
  r := gin.Default()
  limiter := rate.NewLimiter(1000, 2000) // 每秒 1000，桶 2000
  r.GET("/:code", func(c *gin.Context) {
      if !limiter.Allow() { c.Status(429); return }
      code := c.Param("code")
      if url, ok := redis.Get(c, "u:"+code).Result(); ok == nil && url != "" {
          c.Redirect(302, url); return
      }
      url, err := repo.FindByCode(code)
      if err != nil { c.Status(404); return }
      _ = redis.Set(c, "u:"+code, url, 10*time.Minute).Err()
      c.Redirect(302, url)
  })
  ```
* **部署**：K8s（2 份 API pod、1 份 worker、Redis Cluster、Postgres HA），Nginx Ingress + `enable_dos_mitigation`
* **測試**：`go test -run -bench=. -benchtime=2s`、wrk/hey 壓測、pprof flamegraph（CPU / alloc）

---

## 3) 分散式鎖與併發模型對照（Java+Go 雙語）

* **技術棧**：Java（Spring Data + Postgres）、Go（etcd/Redis redsync）、JMH、pprof
* **功能**：同一功能用三種鎖法：DB 悲觀鎖、`pg_try_advisory_xact_lock`、Redis 分散式鎖；比較吞吐與一致性
* **性能要求**：在 1k 併發下比較 P95 延遲與衝突率
* **核心程式碼（Java：Postgres advisory lock 包裝）**

  ```java
  @Transactional
  public void withPgAdvisoryLock(long key, Runnable body) {
    jdbcTemplate.queryForObject("SELECT pg_try_advisory_xact_lock(?)", Boolean.class, key);
    body.run();
  }
  ```
* **部署**：Docker Compose（pg、redis、etcd），Grafana 展板匯出
* **測試**：JMH/JUnit + k6 比較報告（表格型輸出）

---

## 4) 以 Go 實作 Kafka 消費者群組「精準一次」語義

* **技術棧**：Go、`segmentio/kafka-go` 或 `confluent-kafka-go`、Postgres、Outbox/Inbox、OTEL
* **功能**：消費→本地交易寫入 + inbox 去重（consumer 端 exactly-once）
* **性能要求**：每秒處理 5k 訊息、重播 10 萬筆無重複
* **核心程式碼（Go：Inbox 去重）**

  ```go
  func handle(msg kafka.Message) error {
      return withTx(func(tx *sql.Tx) error {
          if exists(tx, msg.Topic, msg.Key, msg.Offset) { return nil }
          if err := applyBusiness(tx, msg.Value); err != nil { return err }
          return insertInbox(tx, msg.Topic, msg.Key, msg.Offset)
      })
  }
  ```
* **部署**：K8s + Kafka（Strimzi/MSK），consumer group `min.insync.replicas=2`
* **測試**：故障注入（kill pod / network delay）、重播一週資料驗證無重入

---

## 5) 搜尋服務（Java）—「Elasticsearch + Query Caching」

* **技術棧**：Spring Boot、Elasticsearch、Redis、Micrometer、JMH
* **功能**：全文檢索、拼音/同義詞、熱門查詢快取、冷查詢回源
* **性能要求**：熱門查詢 P99 < 50ms，冷查詢 < 200ms，命中率 > 70%
* **核心程式碼（Java：快取包裝）**

  ```java
  public SearchResult search(String q){
    String k = "q:"+DigestUtils.sha1Hex(q);
    return cache.get(k, () -> es.search(q), Duration.ofMinutes(5));
  }
  ```

---

## 6) 風控規則引擎（Go）

* **技術棧**：Go、CEL（Common Expression Language）、Redis、Postgres、OTEL
* **功能**：以 CEL 定義規則，熱更新；雙寫審計
* **性能要求**：每規則 1ms 內評估；QPS 2k
* **核心程式碼（Go：CEL 編譯快取）**

  ```go
  type Rule struct{ Expr string }
  var progCache sync.Map
  func eval(rule Rule, vars map[string]interface{}) (bool, error) {
      p, ok := progCache.Load(rule.Expr)
      if !ok {
          ast, _ := cel.Parse(rule.Expr)
          env, _ := cel.NewEnv()
          prg, _ := env.Program(ast)
          progCache.Store(rule.Expr, prg); p = prg
      }
      out, _, err := p.(cel.Program).Eval(vars)
      return out.Value().(bool), err
  }
  ```

---

## 7) 觀測性最佳實踐 Demo（Java+Go）

* **技術棧**：Micrometer / OpenTelemetry、Prometheus、Grafana、Loki、Tempo
* **功能**：為上述任一服務加三本柱（Metrics/Logs/Traces），附現成儀表板
* **性能要求**：指標抓取開銷 < 5% CPU、日誌體積壓縮比 > 60%
* **核心程式碼（Java：Micrometer + 自定義計時器）**

  ```java
  @Component
  public class Metrics {
    private final Timer orderTimer;
    public Metrics(MeterRegistry r){ this.orderTimer = r.timer("order.create"); }
    public <T> T timed(Supplier<T> s){ return orderTimer.record(s::get); }
  }
  ```

---

## 共用：效能基準與指標展示

### 1) JMH（Java）基準測試範例

```java
@State(Scope.Benchmark)
public class JsonBench {
  private ObjectMapper om = new ObjectMapper();
  private String payload = "{\"a\":1,\"b\":\"x\"}";
  @Benchmark public Map<String,Object> parse() throws Exception {
    return om.readValue(payload, new TypeReference<>(){});
  }
}
```

* **執行**：`mvn -DskipTests -Pjmh clean install && java -jar target/benchmarks.jar`
* **展示**：將結果輸出 CSV，畫出 thrpt/op（README 圖＋數據表）

### 2) Go `testing`/`pprof` 基準

```go
func BenchmarkEncode(b *testing.B) {
    for i := 0; i < b.N; i++ { _ = json.Marshal(struct{A int}{1}) }
}
// pprof：go test -bench=. -cpuprofile=cpu.out && go tool pprof cpu.out
```

* **展示**：附 flame graph 截圖、Top N 函式耗時表（README）

### 3) k6 壓測腳本（HTTP API 通用）

```javascript
import http from 'k6/http'; import { sleep, check } from 'k6';
export let options = { stages: [{duration:'30s', target:2000},{duration:'2m', target:2000},{duration:'30s', target:0}] };
export default function () {
  const res = http.post('https://svc/orders', JSON.stringify({sku:"A", qty:1}), { headers: {'Content-Type':'application/json'}});
  check(res, { 'status 2xx': r => r.status >=200 && r.status < 300, 'p95 < 120ms': r => r.timings.duration < 120 });
  sleep(1);
}
```

* **展示**：把 k6 `summary.json` 轉表格（P50/P95/P99、吞吐、失敗率），README 貼圖。

### 4) 指標蒐集與儀表板

* **Java（Micrometer）**：啟用 `management.endpoints.web.exposure.include=prometheus`，Grafana 套用 JVM/HTTP 服務板。
* **Go（OTEL + Prometheus exporter）**：HTTP 延遲 histogram、Redis 命中率 gauge、消費者 lag。
* **README 展示**：

  * 截圖：系統拓撲、Grafana 數據（P95、錯誤率、RPS）
  * 故障演練：關閉一個副本後，RPS 回穩時間（秒）與錯誤率峰值（%）

---

## 履歷撰寫模版（可直接套用）

**專案名稱 – 角色（個人 / 主導） | 期間**

* 用 *Java/Go* 在 *K8s* 上實作 *事件驅動電商*，採 *Outbox+SAGA* 確保最終一致：
* **影響**：穩態 *RPS 800*、**P95 120ms**、錯誤率 **0.4%**；故障注入（broker 掛掉 60s）期間 **資料不重複/不丟失**
* **技術**：Spring Boot / Kafka / Postgres / Debezium / Micrometer / Prometheus / Grafana / k6
* **證據**：PR/Commit 連結、Grafana 截圖、k6 報表、JMH/pprof 數據

---

## 提升合格率的小撇步（面試官視角）

1. **每個 repo 有「一頁式 README」**：架構圖、如何跑、指標圖、壓測腳本與結果表格。
2. **擺一個「/benchmarks」資料夾**：原始 CSV/PNG 截圖與產生指令。
3. **一眼看到「可用性」**：`docker compose up` 能啟動、`make test` 能跑綠。
4. **故事線**：需求 → 設計權衡 → 觀測性 → 故障演練 → 效能結果 → 改善清單。

---

[1]: https://www.kubernetes.dev/docs/guide/first-contribution/?utm_source=chatgpt.com "Making your First Contribution"
[2]: https://github.com/kubernetes/kubernetes/contribute?utm_source=chatgpt.com "Contribute to kubernetes/kubernetes"
[3]: https://github.com/golang/go/issues?utm_source=chatgpt.com "Issues · golang/go"
[4]: https://github.com/golang/go/issues/63866?utm_source=chatgpt.com "failures with `not a Go object file` · Issue #63866 · golang/ ..."
[5]: https://github.com/spring-projects/spring-boot/issues?utm_source=chatgpt.com "Issues · spring-projects/spring-boot"
[6]: https://issues.apache.org/jira/projects/KAFKA/summary?utm_source=chatgpt.com "Kafka - ASF JIRA"
[7]: https://medium.com/%40phuctran3289/how-i-started-my-journey-on-contributing-to-apache-kafka-8a394664dab8?utm_source=chatgpt.com "How I started my journey on contributing to Apache Kafka"
[8]: https://developer.confluent.io/learn-more/podcasts/top-6-worst-apache-kafka-jira-bugs/?utm_source=chatgpt.com "Top 6 Worst Apache Kafka JIRA Bugs"
[9]: https://goodfirstissue.dev/?utm_source=chatgpt.com "Good First Issue: Make your first open-source contribution"
[10]: https://github.com/topics/good-first-issue?utm_source=chatgpt.com "good-first-issue · GitHub Topics"
