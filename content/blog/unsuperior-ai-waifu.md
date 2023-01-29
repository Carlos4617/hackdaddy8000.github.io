---
title: Unsuperior AI Waifu Now Available
date: 2023-01-29 9:43:00
tags: ["ai", "llm", "anime"]
series: ["AI"]
featured: true
---

[Github Repo Here](https://github.com/hackdaddy8000/unsuperior-ai-waifu)

Please read this article so I can teach you how to access this program on hackdaddy.dev

The program requires you to give it a few API keys it needs to work. I made an HTML form that does all the hard work for you.
Don't worry about the API keys. They're easy to get and they give you plenty of free credits.

> Q: What is an API key?
> 
> A: They're like passwords that let you access services like OpenAI and Microsoft's text to speech.

> Q: Why do I need to get my own API keys? Can't you make them for us? 
> 
> A: They're like passwords. I'm not giving you my password. Make your own account.

> Q: Is this a scam to get my API keys?
> 
> A: No. You don't need a credit card to sign up for these websites. The most I could ever steal are your free credits. Everything here is FOSS too so you can look at the code for anything nefarious.

## Features
  * Completely free with given free credits.
  * Easy to run.
  * The mouths of normal VTubers, neurosama move based on the volume of their speech. USAW's mouth moves based off the phonetic sound she's making. Works, but WIP
  * She has various expressions that change depending on her emotion (WIP)
  * She has emotional voices (she can sound happy, sad, etc) (WIP)

## How to Procure an OpenAI API Key

1. Make an OpenAI account
2. Go to [this page](https://beta.openai.com/account/api-keys)
3. Create your API key

## How to procure an Azure Speech Key

1. Make an Azure account
2. [Create a Speech resource in the azure portal](https://portal.azure.com/#create/Microsoft.CognitiveServicesSpeechServices)
3. [Get the API key according to this guide](https://learn.microsoft.com/en-us/azure/cognitive-services/cognitive-services-apis-create-account#get-the-keys-for-your-resource). Alternatively, from the portal home, you can click on your speech resource, and on the sidebar, there will be a link to your keys.
![azure portal](/images/speech-portal-key.jpg)

## Form

Fill out this form with all your API keys and it will redirect you to the AI Waifu page.
The raw code for this page is [here](https://github.com/hackdaddy8000/hackdaddy8000.github.io/blob/master/content/blog/unsuperior-ai-waifu.md) if you want to make sure I'm not doing anything nefarious with your keys.

<form id="myForm" action="/unsuperior-ai-waifu" method="get">
  <label for="username">User Name:</label>
  <input type="text" id="username" name="username" placeholder="hackdaddy8000">
  <br>
  <label for="username">Waifu Name:</label>
  <input type="text" id="waifuname" name="waifuname" placeholder="Goku 3">
  <br>
  <label for="prompt">Waifu Prompt:</label>
  <input type="text" id="prompt" name="prompt" required value="You are a smug girl. You enjoy belittling people. You love pointing out how much smarter you are than everyone. You are currently hiding in your basement from the feds. The feds are after you due to various cyber crimes. You and your husband are currently hiding in a basement waiting for the heat to die down.">
  <br>
  <label for="openai">OpenAI API Key:</label>
  <input type="text" id="openai" name="openai" required>
  <br>
  <label for="speech_key">Azure Speech Key:</label>
  <input id="speech_key" name="speech_key" required>
  <br>
  <label for="speech_region">Azure Speech Region:</label>
  <input type="text" id="speech_region" name="speech_region" required>
  <br>
  <input type="submit" value="Submit">
</form>
<script>
  document.getElementById("myForm").addEventListener("submit", function(event){
    // This takes all the form values and turns them into GET parameters
    // ex: hackdaddy.dev/?GET_PARAM=VALUE
    event.preventDefault();
    var form = event.target;
    var inputs = form.elements;
    var inputs_length = inputs.length;
    var url = form.action + "?";
    for(var i = 0; i < inputs_length; i++){
        if(inputs[i].name != ""){
            url += inputs[i].name + "=" + inputs[i].value + "&";
        }
    }
    url = url.slice(0,-1);
    window.location.href = url;
});
</script>