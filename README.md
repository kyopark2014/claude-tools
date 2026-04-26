# claude-tools

## SKILLs


## MCPs

### Web Fetch 

```java
{
   "mcpServers":{
      "web_fetch":{
         "command":"npx",
         "args":[
            "-y",
            "mcp-server-fetch-typescript"
         ]
      }
   }
}
```

### Tavily

```java
{
   "mcpServers":{
      "tavily-search":{
         "command":"python",
         "args":[
            "~/Documents/src/claude-tools/mcp/mcp_server_tavily.py"
         ]
      }
   }
}
```

### Use AWS

```java
{
   "mcpServers":{
      "use-aws":{
         "command":"python",
         "args":[
            "~/Documents/src/claude-tools/mcp/mcp_server_use_aws.py"
         ]
      }
   }
}
```

### kb-retriever

```java
{
   "mcpServers":{
      "kb_retriever":{
         "command":"python",
         "args":[
            "~/Documents/src/claude-tools/mcp/mcp_server_retrieve.py"
         ]
      }
   }
}
```

### AWS Document


```java
{
   "mcpServers":{
      "awslabs.aws-documentation-mcp-server":{
         "command":"uvx",
         "args":[
            "awslabs.aws-documentation-mcp-server@latest"
         ],
         "env":{
            "FASTMCP_LOG_LEVEL":"ERROR"
         }
      }
   }
}
```

### trade_info

```java
{
   "mcpServers":{
      "trade_info":{
         "command":"python",
         "args":[
            "~/Documents/src/claude-tools/mcp/mcp_server_trade_info.py"
         ]
      }
   }
}
```

### Drawio

```java
{
   "mcpServers":{
      "drawio":{
         "command":"npx",
         "args":[
            "@drawio/mcp"
         ]
      }
   }
}
```

### text_extraction

```java
{
   "mcpServers":{
      "text_extraction":{
         "command":"python",
         "args":[
            "~/Documents/src/claude-tools/mcp/mcp_server_text_extraction.py"
         ]
      }
   }
}
```

### Notion

```java
{
   "mcpServers":{
      "notionApi":{
         "command":"npx",
         "args":[
            "-y",
            "@notionhq/notion-mcp-server"
         ],
         "env":{
            "NOTION_TOKEN":"utils.notion_api_key"
         }
      }
   }
}
```


### Slack


```java
{
   "mcpServers":{
      "slack":{
         "command":"npx",
         "args":[
            "-y",
            "@modelcontextprotocol/server-slack"
         ],
         "env":{
            "SLACK_BOT_TOKEN":"os.environ"[
               "SLACK_BOT_TOKEN"
            ],
            "SLACK_TEAM_ID":"os.environ"[
               "SLACK_TEAM_ID"
            ]
         }
      }
   }
}
```

### Weather

```java
{
   "mcpServers":{
      "korea-weather":{
         "command":"python",
         "args":[
            "~/Documents/src/claude-tools/mcp/mcp_server_korea_weather.py"
         ]
      }
   }
}
```

### image_generation

```java
{
   "mcpServers":{
      "image_generation":{
         "command":"npx",
         "args":[
            "-y",
            "image-generation-mcp"
         ]
      }
   }
}
```

### obsidian

```java
{
   "mcpServers":{
      "obsidian":{
         "command":"npx",
         "args":[
            "-y",
            "obsidian-mcp",
            "~/Documents/memo"
         ]
      }
   }
}
```


### browser-use

```java
{
   "mcpServers":{
      "mcp-browser-use":{
         "command":"python",
         "args":[
            "~/Documents/src/claude-tools/mcp/mcp_server_brower_use.py"
         ]
      }
   }
}
```

### aws_sentral

```java
{
   "mcpServers":{
      "aws_sentral":{
         "command":"~/.toolbox/bin/aws-sentral-mcp",
         "args":[
            
         ]
      }
   }
}
```

### aws_outlook

```java
{
   "mcpServers":{
      "aws_outlook":{
         "command":"~/.toolbox/bin/aws-outlook-mcp",
         "args":[
            
         ],
         "env":{
            "OUTLOOK_MCP_ENABLE_WRITES":"true"
         }
      }
   }
}
```

### aws_slack

```java
{
   "mcpServers":{
      "slack-mcp":{
         "command":"slack-mcp",
         "args":[
            
         ]
      }
   }
}
```

### aws_loop

```java
{
   "mcpServers":{
      "loop-mcp":{
         "command":"loop-mcp",
         "args":[
            
         ]
      }
   }
}
```
