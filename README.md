# Anakin connector for Camunda 8

Use [Anakin](https://anakin.io) in BPMN processes to scrape pages, crawl and map sites, and search the web. Results come back as Markdown or JSON and are stored as process variables.

This is an element template for Camunda's built-in REST connector (`io.camunda:http-json:1`). You don't need to build a JAR or run a custom connector runtime.

## Operations

| Operation | Anakin endpoint | Returns |
|---|---|---|
| Scrape URL | `POST /v1/url-scraper/scrape` | The page, in one request |
| Get scrape result | `GET /v1/url-scraper/{id}` | A scrape by `id` |
| AI search | `POST /v1/search` | Search results |
| Start crawl | `POST /v1/crawl` | `jobId` and `status` |
| Get crawl result | `GET /v1/crawl/{id}` | Status and pages |
| Start site map | `POST /v1/map` | `jobId` and `status` |
| Get site map result | `GET /v1/map/{id}` | Status and links |
| Start agentic search | `POST /v1/agentic-search` | `job_id` and `status` |
| Get agentic search result | `GET /v1/agentic-search/{id}` | Status and structured result |

## Setup

1. Create an API key in the [Anakin dashboard](https://anakin.io/dashboard). See [pricing](https://anakin.io/pricing) for the free tier.
2. Add the key as a connector secret named `ANAKIN_API_KEY`:
   - **SaaS:** [create a connector secret](https://docs.camunda.io/docs/components/console/manage-clusters/manage-secrets/) on your cluster.
   - **Self-Managed:** set the environment variable `SECRET_ANAKIN_API_KEY` on the connector runtime. The 8.9 and 8.10 runtimes we tested reject the variable without the `SECRET_` prefix.
3. Add [`anakin-connector.json`](anakin-connector.json) to your project as a connector template, then publish it. In Desktop Modeler, put the file in your [element templates folder](https://docs.camunda.io/docs/components/modeler/desktop-modeler/element-templates/configuring-templates/).
4. On a service task, choose **Change element** and select **Anakin**. Pick an operation and fill in the inputs.

The default result expression stores the response body in `anakinResult`.

## Long-running jobs

Crawl, site map and agentic search run as background jobs:

1. Run the **Start** operation.
2. Wait with a timer event.
3. Run the matching **Get** operation, using the ID from step 1.
4. Loop until `status` is `completed` or `failed`.

The start responses name the ID differently:

| Started with | Job ID expression |
|---|---|
| Start crawl, Start site map | `=anakinResult.jobId` |
| Start agentic search | `=anakinResult.job_id` |

[`examples/anakin-crawl-polling.bpmn`](examples/anakin-crawl-polling.bpmn) is a working crawl loop. Start it with a `startUrl` variable.

A scrape normally finishes in one request. If it comes back with a status other than `completed`, use **Get scrape result** with `=anakinResult.id`.

## Keep process variables small

A scrape response can include `html` and `cleanedHtml` as well as `markdown`, and a crawl returns every page. Store only what the process needs, for example:

```
={pageMarkdown: response.body.markdown}
```

## Errors

API errors fail the job, which retries and then raises an incident. A wrong key shows `Unauthorized` with `Invalid or inactive API key`. Use **Error expression** to turn a response into a BPMN error instead.

A background job can also finish with `status: "failed"`, so check `status` in your process.

## Compatibility

Tested with self-managed Camunda 8.9.28 (connectors 8.9.19) and 8.10.9 (connectors 8.10.4). The template declares `^8.9`.

## Support

- Connector issues: [GitHub issues](https://github.com/Anakin-Inc/camunda-connector-anakin/issues)
- Anakin API: [docs](https://anakin.io/docs) or support@anakin.io

Camunda doesn't provide support for this connector. See [LICENSE.md](LICENSE.md).
