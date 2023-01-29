---
title: Unsuperior AI Waifu Now Available
date: 2023-01-29 16:43:00
tags: ["ai", "llm", "anime"]
series: ["AI"]
featured: true
---

<form id="myForm" action="/unsuperior-ai-waifu" method="get">
  <label for="username">User Name:</label>
  <input type="text" id="username" name="username" default="hackdaddy8000">
  <br>
  <label for="username">Waifu Name:</label>
  <input type="text" id="waifuname" name="waifuname" default="Goku 3">
  <br>
  <label for="prompt">Waifu Prompt:</label>
  <input type="text" id="prompt" name="prompt" required default="You are a smug girl. You enjoy belittling people. You love pointing out how much smarter you are than everyone. You are currently hiding in your basement from the feds. The feds are after you due to various cyber crimes. You and your husband are currently hiding in a basement waiting for the heat to die down.">
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