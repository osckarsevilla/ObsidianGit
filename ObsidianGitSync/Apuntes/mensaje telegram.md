var message = 
## Aquí pones titulo 
*'el mensaje que quieras enviar';
msg.method = "sendMessage";
msg.payload = {"parse_mode": "markdown", "text": message};

return msg;