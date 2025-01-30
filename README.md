# 🚀 ChatGPT in R: Complete Guide 🌟  

**Welcome to the ultimate guide on integrating OpenAI's ChatGPT into R!**  
This repository provides step-by-step instructions on using ChatGPT in R, including **API integration, R functions, interactive applications, and advanced customization.**  

> ⚡ **Designed for developers, data scientists, and researchers looking to leverage ChatGPT in R.**  

---

## 📌 Table of Contents  

- [📢 Getting Started](#-getting-started)  
- [📦 Installing Required Libraries](#-installing-required-libraries)  
- [🛠️ Integrating ChatGPT in R](#️-integrating-chatgpt-in-r)  
- [📜 Creating Custom ChatGPT Functions](#-creating-custom-chatgpt-functions)  
- [🔄 Enhancing ChatGPT with Context Memory](#-enhancing-chatgpt-with-context-memory)  
- [🖼 Using ChatGPT with Images](#-using-chatgpt-with-images)  
- [🎨 Generating Images with DALL·E](#-generating-images-with-dall·e)  
- [✅ Validating API Key](#-validating-api-key)  
- [📊 GitHub Stats](#-github-stats)  

---

## 📢 Getting Started  

To use ChatGPT in R, you **must obtain an API key** from OpenAI.  

🔹 **Get your API key:** [OpenAI Platform](https://platform.openai.com/)  

```plaintext
sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
⚠ **Keep your API key secure** and do not share it publicly.  

---

## 📦 Installing Required Libraries  

Before integrating ChatGPT in R, install the required libraries:

```R
install.packages("httr")
install.packages("jsonlite")
```

- **`httr`** → Handles HTTP requests for API communication.  
- **`jsonlite`** → Converts R objects to JSON format.  

---

## 🛠️ Integrating ChatGPT in R  

The following R script sends a prompt to ChatGPT and retrieves a response:

```R
library(httr)
library(jsonlite)

apiKey <- "sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
prompt <- "R code to remove duplicates using dplyr. Do not write explanations on replies."

response <- POST(
  url = "https://api.openai.com/v1/chat/completions", 
  add_headers(Authorization = paste("Bearer", apiKey)),
  content_type_json(),
  encode = "json",
  body = list(
    model = "gpt-4o-mini",
    temperature = 1,
    messages = list(list(
      role = "user", 
      content = prompt
    ))
  )
)

cat(content(response)$choices[[1]]$message$content)
```

### 📌 Expected Output  
```R
df %>% distinct()
```

---

## 📜 Creating Custom ChatGPT Functions  

This function wraps the ChatGPT API, making it easy to modify parameters dynamically:

```R
chatGPT <- function(prompt, 
                    modelName = "gpt-4o-mini",
                    temperature = 1,
                    apiKey = Sys.getenv("chatGPT_API_KEY")) {
  
  if(nchar(apiKey)<1) {
    apiKey <- readline("Paste your API key here: ")
    Sys.setenv(chatGPT_API_KEY = apiKey)
  }
  
  response <- POST(
    url = "https://api.openai.com/v1/chat/completions", 
    add_headers(Authorization = paste("Bearer", apiKey)),
    content_type_json(),
    encode = "json",
    body = list(
      model = modelName,
      temperature = temperature,
      messages = list(list(
        role = "user", 
        content = prompt
      ))
    )
  )
  
  if(status_code(response)>200) {
    stop(content(response))
  }
  
  trimws(content(response)$choices[[1]]$message$content)
}

cat(chatGPT("square of 29"))
```

---

## 🔄 Enhancing ChatGPT with Context Memory  

This function **remembers prior conversations**:

```R
chatHistory <- list()

chatGPT <- function(prompt, 
                    modelName = "gpt-4o-mini",
                    temperature = 1,
                    max_tokens = 2048,
                    top_p = 1,
                    apiKey = Sys.getenv("chatGPT_API_KEY")) {
  
  chatHistory <<- append(chatHistory, list(list(role = "user", content = prompt)))
  
  response <- POST(
      url = "https://api.openai.com/v1/chat/completions",
      add_headers("Authorization" = paste("Bearer", apiKey)),
      content_type_json(),
      body = toJSON(list(model = modelName, messages = chatHistory), auto_unbox = TRUE)
  )
  
  answer <- trimws(content(response)$choices[[1]]$message$content)
  chatHistory <<- append(chatHistory, list(list(role = "assistant", content = answer)))

  return(answer)
}

cat(chatGPT("2+2"))
cat(chatGPT("square of it"))
cat(chatGPT("add 3 to result"))
```

---

## 🖼 Using ChatGPT with Images  

ChatGPT-4o supports **image input and analysis**:

```R
chatGPT_img <- function(prompt, 
                    image_url,
                    modelName = "gpt-4o",
                    detail = "low",
                    apiKey = Sys.getenv("chatGPT_API_KEY")) {
  
  response <- POST(
    url = "https://api.openai.com/v1/chat/completions", 
    add_headers(Authorization = paste("Bearer", apiKey)),
    content_type_json(),
    encode = "json",
    body = list(
      model = modelName,
      messages = list(
        list(
          role = "user",
          content = list(
            list(type = "text", text = prompt),
            list(
              type = "image_url",
              image_url = list(url = image_url, detail = detail)
            )
          )
        )
      )
    )
  )
  
  trimws(content(response)$choices[[1]]$message$content)
}
```

---

## 🎨 Generating Images with DALL·E  

Create AI-generated images using DALL·E:

```R
chatGPT_image <- function(prompt, 
                    n = 1,
                    size = "1024x1024",
                    response_format = "url",
                    apiKey = Sys.getenv("chatGPT_API_KEY")) {
  
  response <- POST(
    url = "https://api.openai.com/v1/images/generations", 
    add_headers(Authorization = paste("Bearer", apiKey)),
    content_type_json(),
    encode = "json",
    body = list(
      prompt = prompt,
      n = n,
      size = size,
      response_format = response_format
    )
  )
  
  parsed <- jsonlite::fromJSON(httr::content(response, as = "text", encoding = "UTF-8"), flatten = TRUE)
  parsed
}

img <- chatGPT_image("a futuristic city skyline at sunset")
img$data$url
```

---

## ✅ Validating API Key  

```R
apiCheck <- function(apiKey = Sys.getenv("chatGPT_API_KEY")) {
  x <- httr::GET(
    "https://api.openai.com/v1/models",
    httr::add_headers(Authorization = paste0("Bearer ", apiKey))
  )
  
  if (httr::status_code(x) == 200) {
    message("Correct API Key! ✅")
  } else {
    stop("Incorrect API Key ❌")
  }
}
apiCheck()
```

---

## 📊 GitHub Stats  

![GitHub Stats](https://github-readme-stats.vercel.app/api?username=Zaoqu-Liu&show_icons=true&theme=radical)

---

🚀 **Star this repo if you found it useful!** ⭐  
