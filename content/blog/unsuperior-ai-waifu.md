---
title: Unsuperior AI Waifu Now Available
date: 2023-01-29 9:43:00
tags: ["ai", "llm", "anime"]
series: ["AI"]
featured: true
---

[Github Repo Here](https://github.com/hackdaddy8000/unsuperior-ai-waifu)

![Demo](/images/usaw-demo.gif)

Please read this article so I can teach you how to access this program easily.

## Features
  * Completely free with given free credits.
  * Easy to run.
  * You can interact with her by poking her. I wouldn't recommend it though.
  * The mouths of normal VTubers, neurosama move based on the volume of their speech. USAW's mouth movements are more accurate because they're based off the phonetic sound she's making. Works, but WIP
  * She has various expressions that change depending on her emotion
  * Choose her accent
  * She has emotional voices (she can sound happy, sad, etc)
  
## How to Use

You can download [these files](https://github.com/hackdaddy8000/unsuperior-ai-waifu) and open index.html in your web browser. No installation needed. Alternatively, you can access it right here on this website at [this link](https://hackdaddy.dev/unsuperior-ai-waifu).

*** You need to input a few values first or else it won't work.

The program requires you to give it a few API keys that it uses to run. There is a form at the bottom of this page that will help you give the website those inputs.
Don't worry about the API keys. They're easy to get and they give you plenty of free credits.

> Q: What is an API key?
> 
> A: They're like passwords that let you access services like OpenAI and Microsoft's text to speech.

> Q: Why do I need to get my own API keys? Can't you make them for us? 
> 
> A: They're like passwords. I'm not giving you my password. Make your own account.

> Q: Is this a scam to get my API keys?
> 
> A: No. There's nothing I can really take from you. You don't need a credit card to sign up for these websites. The most I could ever steal are your free credits. Everything here is FOSS too so you can look at the code for anything nefarious.

### How to Procure an OpenAI API Key

> Q: Why do I need this?
>
> A: This program uses OpenAI's software to create her dialogue. Without this API key, she is literally a husk. She has no soul.

1. Make an OpenAI account
2. Go to [this page](https://beta.openai.com/account/api-keys)
3. Create your API key

### How to Procure an Azure Speech Key

> Q: Why do I need this?
>
> A: This gives her the ability to hear you, and speak.

> Q: Why didn't you just use the [Speech-to-Text](https://developer.mozilla.org/en-US/docs/Web/API/Web_Speech_API/Using_the_Web_Speech_API) and [Text-to-Speech](https://developer.mozilla.org/en-US/docs/Web/API/SpeechSynthesisUtterance) APIs built into the browser, so that I don't have to bother with making an Azure account?
>
> A: In my experimentation, I could not get StT to work well (or at all) on mobile. That's probabably just a skill issue on my part. Furthermore, Azure TSS is simply better IMO. It has high quality voices, each with different "styles" that make her voice very emotive. The Azure API also tells me what the shape of her mouth should look like as she talks, allowing me to animate her more realistically.

1. Make an Azure account
2. [Create a Speech resource in the azure portal](https://portal.azure.com/#create/Microsoft.CognitiveServicesSpeechServices)
3. [Get the API key according to this guide](https://learn.microsoft.com/en-us/azure/cognitive-services/cognitive-services-apis-create-account#get-the-keys-for-your-resource). Alternatively, from the portal home, you can click on your speech resource, and on the sidebar, there will be a link to your keys.
![azure portal](/images/speech-portal-key.jpg)

## Form

Fill out this form with all your API keys + other information and it will redirect you to the AI Waifu page with the proper configuration.

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
  <label for="voice">Choose a voice:</label>
  <select id="voice" name="voice">
    <option value="en-US-JennyNeural">Jenny (American)</option>
    <option value="en-US-JaneNeural">Jane (American)</option>
    <option value="zh-CN-XiaoxiaoNeural">Xiaoxiao (Chinese)</option>
    <option value="ja-JP-NanamiNeural">Nanami (Japanese)</option>
    <option value="en-US-GuyNeural">Guy (American)</option>
  </select>
  <br>
  <input type="submit" value="Submit">
</form>
<script>
  document.getElementById("myForm").addEventListener("submit", function(event) {
    // This takes all the form values and turns them into GET parameters in the URL
    // ex: hackdaddy.dev/?GET_PARAM1=VALUE&GET_PARAM2=VALUE2
    event.preventDefault();
    var form = event.target;
    var inputs = form.elements;
    var inputs_length = inputs.length;
    var url = form.action + "?";
    for(var i = 0; i < inputs_length; i++) {
        if(inputs[i].name != ""){
            url += inputs[i].name + "=" + inputs[i].value + "&";
        }
    }
    url = url.slice(0,-1);
    window.location.href = url;
});
</script>