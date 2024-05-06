# Scope

## Not in scope

- Setting up NPS
    - There are many guides online -- [this one](https://community.ruckuswireless.com/t5/RUCKUS-Self-Help/802-1x-authentication-with-NPS-policies-Windows-Server-2016/m-p/62773){:target="_blank"} is not bad
- Setting up Graylog
	- There are many guides online -- the [official docs](https://go2docs.graylog.org/5-2/downloading_and_installing_graylog/installing_graylog.html){:target="_blank"} also cover it in detail
	- Basic Graylog architecture:
		``` mermaid
		graph LR
		messages("Log Messages") --> graylog("Graylog Server")
		subgraph os ["OpenSearch Cluster"]
			direction LR
			os1("OpenSearch 1")
			os2("OpenSearch 2")
			osn("OpenSearch 𝒏")
		end
		graylog <--> www("Web Server")
		graylog <--> db("MongoDB")
		graylog <--> os
		```
	- To learn more about IU13's Graylog cluster, attend Brian Steigauf's presentation here at 2:30 pm

## Goals

- [ ] **Configure**: configure NPS to log to disk
- [ ] **Ingest**: use Graylog Sidecar filebeat collector to read the log files and send messages to Graylog
- [ ] **Process**: parse log messages using Graylog Extractors and Pipelines
- [ ] **Aggregate**: make log data useful for meaningful viewing

&nbsp; :medal: **Takeaway**: be able to apply a similar process to other sources of log messages and make use of more data from your environment

## NPS Log Processing Architecture

``` mermaid
graph LR
  subgraph gl ["Graylog Server"]
    graylogapi("Graylog API")
    grayloginput("Graylog Input")
    pipeline("Pipeline
    Processing")
    subgraph extractors ["Extractors"]
      direction BT
      regex("Regex/Grok Patterns")
      lut("Lookup Tables")
      da("Data Adapters")
    end
    logs("Processed Logs")
  end

  subgraph nps ["NPS Server"]
    sidecar("Sidecar Service")
    npsservice("NPS Service")
    file("XML Log File")
  end

  sidecar -- Log messages ---> grayloginput;
  sidecar -- Get config --> graylogapi;
  sidecar -- Read --> file;
  grayloginput --> extractors -- Stream --> pipeline
  pipeline --> logs;
  npsservice -- Write --> file;
  regex -. (when needed) .-> lut;
  lut --> da;
```

