# Azure Prompt Flow Workshop

This repository provides the resources and instructions for a hands-on workshop focused on using Azure Prompt Flow to create and modify flows that turn beautiful poems into the names of the cities that inspired them, and vice versa. The workshop is designed to guide participants through the process of building a basic flow, extending it with additional requirements, and evaluating the flow's performance.

## Purpose

The purpose of this repository is to give participants access to a basic flow in Azure Prompt Flow through a couple of files, including a YAML configuration file. The repository supports a workshop that walks participants through creating and refining flows in Azure Prompt Flow.

## Workshop Steps

### 1. Set Up Your Environment

Before you begin, ensure you have the following set up:

- **VS Code**: Install Visual Studio Code.
- **Prompt Flow Extension**: Install the Azure Prompt Flow extension for VS Code.
- **GitHub Repo**: Clone the GBB-workshop-prompt-flow directory from GitHub.
```
git clone URL_GOES_HERE
```
- **Environment**: Initiate powershell in VS Code and set up a temporary coding environment with conda.
- **Dependencies**: Install required libraries in requirements.txt. First navigate to the **GBB-workshop-prompt-flow** directory in VS Code Explorer. Then enter
```
pip install -r equirements.txt
```
in your conda environment.

### 2. Prepare Azure OpenAI Resources

- **Azure OpenAI Resource**: Using your UI of choice (e.g. the Azure Portal), create an Azure OpenAI resource. Make a note of the endpoint URL and API key for your AOAI resource.
- **OpenAI Model Deployment**: Create a deployment of GPT-4o. Make a note of the name of your deployment, as you will need it soon.

### 3. Connect to your Azure OpenAI Resource

The **GBB-workshop-prompt-flow** repo comes with a basic flow preconfigured for you, with a single LLM node whose prompt is stored in **my_prompt.jinja2**.

A convenient UI for viewing your whole flow can be accessed by navigating to **flow.dag.yaml** in Explorer. Once you're here, click "ctrl+K" and then "V" to open up the flow UI. You should be able to view your flow. The basic flow consists of an input, an output, and an LLM tool that transforms the input into what will become the output.

Before we can use this LLM tool we need to connect it to a Azure OpenAI resource.

Open the Prompt Flow extension for VS Code (it should be visible in your application as a stylized Prompt Flow "P" icon on the left panel). Under "Connections" look for "Azure OpenAI". Click the "+" and fill out the new configuration file given the information from your AOAI resource. You should only have to modify the "name" value; it will then prompt you for the "api_key" value when you create the connection (Read more about setting up Prompt Flow connections [here](https://microsoft.github.io/promptflow/how-to-guides/manage-connections.html)).

After creating your connection, head back to the flow view and scroll down to your LLM node. You can now choose (1) your OpenAI connection, (2) that you want to use the "chat" API, and (3) your model via the deployment name that you gave to it. You should now have a functional 

### Part 1: Use a Basic Flow

You should now have the basic flow ready to go.

Write a good prompt in **my_prompt.jinja2** that asks the LLM to take text (you can use {{poem}} as a variable in your prompt) and make a judgement about what famous city inspired this poem.

The output of your flow should be a single city name, followed by the country (if it's not in the US) or the state abbreviation (if it's in the US); e.g. your output would look something like "Topeka, KS" or "Cardiff, United Kingdom".

Run your flow with the run button in the flow.dag.yaml interface.
You can use the following poem as a sample input to test it:
```
"Longhorns roam under a Texan sky,/nwhile campus smiles at the river flowing by."
```

If you want, go ahead and run a batch of poems using `city_poems.jsonl` as input data.
- Click batch run in the PF interface
- Choose "Local file" and select `city_poems.jsonl`
- In the run config file, under "column_mapping", make sure to set `city_name: ${data.city_name}` (assuming the "data" value is set to the .jsonl file name) to identify the right value.
- Click Run, and navigate to the output (a link should pop up in your powershell terminal)

### Part 2: Evaluate Your Flow

So you think that your new solution is so great at identifying what cities inspired certain poems that AI will soon replace tenured faculty in Art History and English departments.

Let's test this by evaluating our flow. We will run a batch of poems as input and then use a PF flow to compare the city names outputted with the ground truth right answers.

Before we begin, take a look at the `eval_flow` directory that comes included. Everything you need to run this evaluation (except the prediction data itself) is in here. The logic for this flow is simple: if our flow outputs a city name that perfectly matches what's in our ground truth data, it's Correct; otherwise it's Incorrect. You can find this logic implemented in line 18 of the Python file `line_process.py`. If you want, check it out to make sure you agree with it and that it will not produce any false positives or false negatives. Some flexibility with upper-/lowercase is built in; otherwise, everything needs to match.

Now let's run our evaluation.

- First, we need some output to evaluate. If you haven't yet done a batch run with the `city_poems.jsonl` (which contains a set of 30 city-inspired poems), go back to `flow.dag.yaml` in the main GBB-workshop-prompt-flow directory and do that now. (See end of Part 1 for steps.)

Now we have a fresh run to work with.
- Open the .yaml file in the *eval_flow* directory.
- Look for the grey text at the bottom and click "Batch run."
- When you are prompted, say that you want to use an "Existing Run".
- Wait a couple seconds for it to retrieve the most recent ones. Choose the most recent one, which is on top (if you just ran the batch of `city_poems.jsonl`).

Now we have to fill out another run configuration, this time for the evaluation flow. Note that the name of the directory where the data from your previous run is stored is the value for **run**. We need to fill out some other stuff.
- The **run** directory will give us the prediction values but our ground truth values are stored somewhere else. Uncomment (delete the '#' before) the **data** variable, and type the full path of the file `ground_truth.jsonl`. (You should be able to take the value of the **flow** variable, remove *eval_flow* at the end, and add *ground_truth.jsonl* instead. It might look something like `c:\Users\{username}\Documents\GBB-workshop-prompt-flow\ground_truth.jsonl`.)

Under *column_mapping*, make sure that the eval can find both the city names predicted from your run and the correct city names in the ground truth file.

It should look like this:
- `**groundtruth**: ${data.city_name}`
- `**prediction**: ${run.outputs.city_name}`

Save the run configuration file and then *Run* it.

After the eval is complete, in the terminal you will find a link to the eval output. It might look something like `C:/Users/{uername}/.promptflow/.runs/eval_flow_default_20240829_160401_372000/flow_outputs`.
- You can navigate here to see the eval results line by line.
- From the *flow_outputs* directory, you can navigate up one level to find other useful files such as `metrics.jsonl`, which will tell you the overall accuracy of your flow based on the aggregated results of this evaluation run.

### Part 3: Create a New Flow

Now that you have successfully created a flow that can detect what cities inspired these beautiful poems with a known degree of success, let's create a new flow.

Your repo has a mostly empty directory called **generate-poem-flow**. Right-click it in the VS Code Explorer and then click "New flow in this directory" and choose "Standard flow with the template" when prompted. The Prompt Flow extension will automate the creation of a new bare-bones flow in this directory.

Set up the OpenAI tool to use your currently existing AOAI connection.

The prompt will be similar to the one in Part 1 except in reverse: the task in this step is to build a flow that takes the name of a famous world (or US) city as input and give a short poem about the city as output.

### Part 4: Add a Seasonal Element to the Poem

Our poems are good but they are too abstract. We need to add some concrete real-world information to make them more up-to-date.

The new requirement is that the outputted poems about world cities reference *the current season* to make them immediately relevant for a reader who might want to travel to that city soon. This means that you'll need to give your prompt two pieces of information instead of one. In addition to knowing the *city_name*, we will also need to somehow retrieve the *current_season* and then feed that to the LLM node as input.

To do this, we'll add a Python node that retrieves the current date and time via NTP, then parses out the season.

The Python code itself is already written for you. You just need to:

- create a new Python tool (node) in the Prompt Flow interface ("+ Python")
- name the new node something that makes sense, e.g. "get_season"
- select that we are importing an "Existing file" and point it to *get_current_season.py*

Now that we know the current season as a string output from our Python tool, we can inject it into our prompt.

- Make sure that the LLM node now has $(your python node).output as one of its inputs. You can call the variable whatever you want (like *current_season*) but note that you need to call it the exact same thing in the jinja2 file where you have your prompt.
- Augment that jinja2 file to ask the LLM to generate a poem based not just on a city but also the current season, and let it know (e.g. `{{current_season}}`) what the current season is.

Try it out using a sample city.

If you want, you can even run a batch of city names using `cities.jsonl` as input data.
- Click batch run in the PF interface
- Choose "Local file" and select `cities.jsonl`
- In the run config file, under "column_mapping", make sure to set `city_name: ${cities.city_name}` to identify the right value.
- Click Run and navigate to the output (a link should pop up in your powershell terminal)

### Part 5: Include Information About Activities in the City

Your poem-generation app has been wildly successful and it was purchased by an international travel agency. They want the poem to include one more thing: information about fun things to do or visit when vacationing as a tourist in that city.

Data about touristy activities for many major world cities already exists as a .csv file  You will need to connect to a blob storage resource that contains 
In this part, you will further extend the flow by adding another requirement: the poem must include information about fun things to do in the city.

- NEED TO FINISH THIS DEPENDING ON HOW WE DECIDE TO HANDLE THE BLOB STORAGE CONNECTION

*(Remember how city names are formed in the dataset: non-US countries are spelled out, and US states are abbreviated. Searching the database for "Paris" will fail to find "Paris, France" and searching for "New York City" will fail to find "New York, NY".)*

## Conclusion

By the end of this workshop, you now have hands-on experience building and modifying flows in Azure Prompt Flow, and you know how to evaluate their performance using a testing dataset.