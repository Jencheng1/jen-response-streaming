# jen-response-streaming

In this lab, we will build a response streaming application using Bedrock, LangChain, and Streamlit.

Streaming responses are useful when you want to start returning content immediately to the end user. You can display the output a few words at a time, instead of waiting for the entire response to be created.

Use cases
The response streaming pattern is good for the following use cases:

Situations where longer text will be generated, but you want to keep the user engaged by beginning to return a response immediately.
Architecture
Architecture diagram, illustrating the introduction above

From an architectural perspective, response streaming is similar to text-to-text. We just need to add a special handler to immediately process the streaming output as it is created.

The streamed response is returned in chunks of JSON. You can then extract the returned text from each chunk to be displayed to the end user.

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

 

**Create the library script**

First, we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

Navigate to the workshop/labs/streaming folder, and open the file streaming_lib.py

Add the import statements.

These statements allow us to use LangChain to call Bedrock and process the streaming output.

from langchain.chains import ConversationChain
from langchain.llms import Bedrock


Add the code to create a LangChain Bedrock client with streaming enabled.

   def get_llm(streaming_callback):
    model_kwargs = {
        "max_tokens": 4000,
        "temperature": 0,
        "p": 0.01,
        "k": 0,
        "stop_sequences": [],
        "return_likelihoods": "NONE",
        "stream": True
    }
    
    llm = Bedrock(
        model_id="cohere.command-text-v14",
        model_kwargs=model_kwargs,
        streaming=True,
        callbacks=[streaming_callback],
    )
    
    return llm

 

Add this function to call Bedrock and handle the streaming response.

This function calls Bedrock with the streaming invocation method. As response chunks are returned, it passes the chunk's text to the provided callback method.
def get_streaming_response(prompt, streaming_callback):
    conversation_with_summary = ConversationChain(
        llm=get_llm(streaming_callback)
    )
    return conversation_with_summary.predict(input=prompt)



Save the file.
Excellent! You are done with the backing library. Now we will create the front-end application.

 

**Create the Streamlit front-end app**


In the same folder as your lib file, open the file streaming_app.py
 

Add the import statements.

These statements allow us to use Streamlit elements and call functions in the backing library script.
import streaming_lib as glib  # reference to local lib script
import streamlit as st
from langchain.callbacks import StreamlitCallbackHandler


 

Add the page title, configuration, and two-column layout.

Here we are setting the page title on the actual page and the title shown in the browser tab.
st.set_page_config(page_title="Response Streaming")  # HTML title
st.title("Response Streaming")  # page title


 

Add the input elements.

We are creating a multiline text box and button to get the user's prompt and send it to Bedrock.
input_text = st.text_area("Input text", label_visibility="collapsed")
go_button = st.button("Go", type="primary")  # display a primary button


 

Add the output elements.

We use the if block below to handle the button click.
We create an empty streamlit container and pass it to the StreamlitCallbackHandler object so it can display output as it is generated.
We pass the StreamlitCallbackHandler to the backing function so it can handle responses as streaming chunks are returened.
if go_button:  # code in this if block will be run when the button is clicked
    #use an empty container for streaming output
    st_callback = StreamlitCallbackHandler(st.container())
    streaming_response = glib.get_streaming_response(prompt=input_text, streaming_callback=st_callback)



 

Save the file.
Superb! Now you are ready to run the application!

 

**Run the Streamlit app** 

Select the bash terminal in AWS Cloud9 and change directory.
cd ~/environment/workshop/labs/streaming

Just want to run the app?
Expand here & run this command instead
 

Run the streamlit command from the terminal.
streamlit run streaming_app.py --server.port 8080

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

 

In AWS Cloud9, select Preview -> Preview Running Application.
Screenshot of terminal, showing the Cloud9 preview button

You should see a web page like below:

Streamlit app at launch

 

Try out some prompts and see the results.

Write a story about two cats that go on an adventure:
Streamlit app in use

 

Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.
