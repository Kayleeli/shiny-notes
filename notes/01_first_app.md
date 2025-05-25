# Introduction

All notes were complied from the book: <https://mastering-shiny.org/index.html>

## Installation

```R
install.packages("shiny")
library(shiny)
```

## Create Boilerplate

- Comments below explain each part of the app structure.
- Two ways to generate the boilerplate:
  - Manually: Create a directory, add an app.R file, type shinyapp in the file and press Shift + Tab.
  - GUI method: Go to File ▸ New Project ▸ New Directory ▸ Shiny Web Application.

```R
# loads the shiny package
library(shiny)

# define UI 
# fluidPage() create responsive layout that adjust to screen size
# UI elements are placed inside fluidPage
ui <- fluidPage(
  
)

# define server logic (aka, how app react to user input)
# input: contains user input (i.e. slider value, text-input)
# output: use to sent data from server back to UI (e.g. plot, tables)
# session: give control over user's session (optional)
server <- function(input, output, session) {
  
}

# launch the app by combine UI + server 
shinyApp(ui, server)
```

## UI controls

| Function               | Type            | Purpose                                                      |
| ---------------------- | --------------- | ------------------------------------------------------------ |
| `fluidPage()`          | Layout Function | Sets up basic structure and layout of the app UI             |

| Function (input)       | Type            | Purpose                                                      |
| ---------------------- | --------------- | ------------------------------------------------------------ |
| `selectInput()`        | Input  Control  | Creates a dropdown menu for selecting a built-in dataset     |
| `sliderInput()`        | Input Control   | Create slider input                                          |

| Function (output)      | Type            | Purpose                                                      |
| ---------------------- | --------------- | ------------------------------------------------------------ |
| `verbatimTextOutput()` | Output Control  | Displays console-style output (e.g., result of `summary()`)  |
| `textOutput()`         | Output Control  | Displays plain text (for clean, short messages)              |
| `tableOutput()`        | Output Control  | Displays data tables (e.g., the selected dataset’s contents) |
| `plotOutput()`         | Output Control  | Displays plot                                                |

 
### UI Controls Detail Breakdown

#### `selectInput()`: Drop-down Menu

```r
# Creates a drop-down input for selecting a value.

selectInput("dataset", label = "Dataset", choices = ls("package:datasets"))

# `"dataset"`                       : the **input ID** (used in the server to access user selection)
# `label = "Dataset"`               : the **label** shown next to the drop-down
# `choices = ls("package:datasets")`: the **options** shown in the drop-down (all built-in datasets in R)
```

#### `sliderInput()`: Horizontal Slider

```r
# Create horizontal slider for selecting a value.

sliderInput("x", label = "If x is", min = 1, max = 50, value = 30)

# "x"                : input ID used for reference the slider value
# label = "If x is"  : set the label beside the slider
# min = 1, max = 50  :  define the range
# value - 30         :  define default starting value
# "then x times 5 is": A static text string shown in the UI.
# textOutput("product"): A placeholder for displaying the computed result
```

#### `textOutput()`: Text Output Box (plain style)

```r
# Displays plain text output (e.g., printed summary or console-style output).

  textOutput("product")

# product: refer to the ID that links the UI component to a specific output in the server (user-defined, can be anything, but gotta be consistent
```


#### `verbatimTextOutput()`: Text Output Box (console style)

```r
# Displays plain text output (e.g., printed summary or console-style output).

verbatimTextOutput("summary")

# `"summary"`: the **output ID**
#      Used in the server to send output (e.g., `summary()` result) to this spot in the UI
```

#### `tableOutput()`: Data Table Display

```r
# Displays tabular data.

tableOutput("table")

# `"table"`: the output ID
#   Used in the server to send a dataset or data frame to be shown as a table in the UI
```

#### `plotOutput()`: Plot

```r
# Displays plot

plotOutput("plot")

# `"plot"`: the output ID
#   Used in the server to send a dataset or data frame to be shown as a plot in the UI
```

## Behaviour

| Input/Output     | Explanation                                                      |
| ---------------- | ---------------------------------------------------------------- |
| `input$dataset`  | The current selection from the drop-down menu                    |
| `output$summary` | The result printed in the text output area                       |
| `output$table`   | The data table shown in the table output area                    |

| Renders          | Explanation                                                      |
| ---------------- | ---------------------------------------------------------------- |
| `renderPrint()`  | Tells Shiny how to print text like the result of `summary()`     |
| `renderTable()`  | Tells Shiny how to display a table from a data frame             |
| `renderText()`   | Tell shiny how to render out text                                |
| `renderPlot()`   | Tell shiny how to render out plot                                |
| Reactivity       | Outputs automatically respond to input changes                   |

### Behaviour Detailed Breakdown

```r
# defines how it respond to user action
server <- function(input,output,function) {
...
# input = hold what user picked/typed
# output = where we show results
# session = manage app behind the scene
}
```

```r
# when someone select a dataset: (1) find the dataset (2) go find the datset (3) show summary
  output$summary <- renderPrint({
    dataset <- get(input$dataset, "package:datasets")
    summary(dataset)
  })
# renderPrint() : tell shiny to print plain text
# get(): fetch dataset user picked
# summary(...): function that description of dataset
```

```r
# when someone select a dataset: (1) show full table of that dataset
  output$table <- renderTable({
    dataset <- get(input$dataset, "package:datasets")
    dataset
  })
# renderTable(): make dataset show as table
```

```r
# when someone select a dataset: (1) show plot of that dataset
  output$plot <- renderPlot({
    plot(dataset())  
  }, res = 96)
}
# renderPlot(): make dataset show as plots matrix
# res=96 : set resolution of the plot
```

### Reactive Expression (avoid repeating lines)

- `dataset <- get(input$dataset, "package:datasets")` tends to repeat (because we load the dataset based on user input)
- use `reactive()` instead - it's like storing this line as a reusable box

Example:

```r
server <- function(input, output, session) {
  
  # reusable box
  dataset <- reactive({
    get(input$dataset, "package:datasets") 
    # package:datasets tell R to look inside the dataset pakcage environment for the object. 
    # Without, R would look for the object in  global env, which might cause ambiguity/error.
  })

  output$summary <- renderPrint({
    summary(dataset())  # use the result from the box
  })
  
  output$table <- renderTable({
    dataset()  # use the same box again
  })
}
```

## Running App

### Ways to Launch the App

- Click the **Run App** button in the RStudio toolbar
- Or press **Cmd/Ctrl + Shift + Enter**

### App url format

- The app runs on a local server using a random port
- Console output example: `Listening on http://127.0.0.1:3827`
  - `127.0.0.1` = localhost (your own computer)
  - `3827` = randomly assigned port number for that session
