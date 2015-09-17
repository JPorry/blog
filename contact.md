---
layout: page
title: Contact
permalink: /contact/
---

<p>Please, feel free to send me a message if I can do something for you or if you have any ideas that can make this humble blog a little bit better.</p>

<p id="thanksMessage" class="success">Thanks a lot for that message!! Keep it up!</p>
<p id="errorMessage" class="error">Oooops! Looks like something went wrong sending the email. Try again, please!</p>

<form id="contactForm" action="http://formspree.io/porrero.javier@gmail.com" method='POST'>
  <input id="emailField" type="email" name="_replyto" placeholder="Email">
  <textarea name="body" placeholder="Tell me something..."></textarea>
  <input type="hidden" name="_next" value="http://localhost:8000/contact/" />
  <input id="sendButton" type="submit" value="Send">
</form>


<script>
  (function(){
    var form = document.getElementById('contactForm');
    var btn = document.getElementById('sendButton');
    var errorMessage = document.getElementById('errorMessage');
    var thanksMessage = document.getElementById('thanksMessage');

    

    form.addEventListener('submit', function(e){
      btn.disabled = true;
      e.preventDefault();
      var request = new XMLHttpRequest();

      request.open('POST', form.action);

      request.setRequestHeader('accept', 'application/json');
      request.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');

      request.onload = function(){
        if(request.status === 200){
          thanksMessage.style.display = 'block';
          errorMessage.style.display = 'none';
        }else{
          thanksMessage.style.display = 'none';
          errorMessage.style.display = 'block';
        };
        btn.disabled = false;
      }

      request.onerror = function(){
        thanksMessage.style.display = 'none';
        errorMessage.style.display = 'block';
        btn.disabled = false;
      }

      request.send(
        "email=" + form.querySelector('#emailField').value +
        "&message=" + form.querySelector('textarea').value);
    });
    
  })()
</script>