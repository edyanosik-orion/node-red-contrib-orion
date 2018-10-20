Read your tone
--------------

# Speech To Text and Tone Analyzer with IBM Watson
The following flow uses [IBM Watson Speech to Text](https://www.ibm.com/watson/services/speech-to-text/) to convert Orion PTT Audio to text, and [IBM Watson Tone Analyzer](https://www.ibm.com/watson/services/tone-analyzer/) to identify the tone of your message. Tone scores are ranked and the top result is returned in a response PTT over Orion.  If no tone could be identified, a friendly error message is sent over Orion.

## Requirements

This flow requires an IBM Watson Speech-To-Text service account and IBM Watson Tone Analyzer and associated app keys. IBM has a "Lite" account which is free.

## Orion Flow

Coming Soon

## Code for Importing into Node-RED or Orion Aster

```json
[{"id":"ddb7a6e2.286ce","type":"tab","label":"Flow 2","disabled":false,"info":""},{"id":"622acd5c.aa7d7c","type":"orion_rx","z":"ddb7a6e2.286ce","name":"IBM RX","orion_config":"1eb9a6af.e955e9","x":90,"y":220,"wires":[["c8286cfc.65c69"]]},{"id":"8b5da2c8.80e34","type":"orion_decode","z":"ddb7a6e2.286ce","name":"Decode media","x":660,"y":220,"wires":[["7088fe0.0781104"]],"inputLabels":["msg.media"],"outputLabels":["msg.payload"]},{"id":"c8286cfc.65c69","type":"switch","z":"ddb7a6e2.286ce","name":"Only PTT","property":"event_type","propertyType":"msg","rules":[{"t":"eq","v":"ptt","vt":"str"}],"checkall":"true","repair":false,"outputs":1,"x":280,"y":220,"wires":[["53fe8094.52"]]},{"id":"7088fe0.0781104","type":"change","z":"ddb7a6e2.286ce","name":"","rules":[{"t":"set","p":"payload","pt":"msg","to":"media_wav","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":380,"y":300,"wires":[["6326bd9c.5f0294","f7ac055d.fd4d48"]]},{"id":"6326bd9c.5f0294","type":"watson-speech-to-text","z":"ddb7a6e2.286ce","name":"","alternatives":1,"speakerlabels":true,"smartformatting":false,"lang":"en-US","langhidden":"en-US","langcustom":"NoCustomisationSetting","langcustomhidden":"","band":"NarrowbandModel","bandhidden":"","password":"NmbA1Oy8jvqd","apikey":"","payload-response":false,"streaming-mode":false,"streaming-mute":true,"auto-connect":false,"discard-listening":false,"disable-precheck":true,"default-endpoint":false,"service-endpoint":"https://stream.watsonplatform.net/speech-to-text/api","x":140,"y":380,"wires":[["b9ad48e3.3a94e"]]},{"id":"40f862a1.41c7e4","type":"watson-tone-analyzer-v3","z":"ddb7a6e2.286ce","name":"","tones":"emotion","sentences":"true","contentType":"false","tone-method":"generalTone","interface-version":"2016-05-19","inputlang":"en","default-endpoint":true,"service-endpoint":"https://gateway.watsonplatform.net/tone-analyzer/api","x":560,"y":380,"wires":[["60168ea5.7d841"]],"inputLabels":["msg.transcription"]},{"id":"b9ad48e3.3a94e","type":"change","z":"ddb7a6e2.286ce","name":"","rules":[{"t":"set","p":"payload","pt":"msg","to":"transcription","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":340,"y":380,"wires":[["40f862a1.41c7e4"]]},{"id":"60168ea5.7d841","type":"sort","z":"ddb7a6e2.286ce","name":"","order":"ascending","as_num":true,"target":"response.document_tone.tone_categories[0].tones","targetType":"msg","msgKey":"score","msgKeyType":"jsonata","seqKey":"payload","seqKeyType":"msg","x":130,"y":500,"wires":[["951d95d9.b3eda"]]},{"id":"951d95d9.b3eda","type":"change","z":"ddb7a6e2.286ce","name":"Identify top tone","rules":[{"t":"set","p":"highest_tone","pt":"msg","to":"response.document_tone.tone_categories[0].tones[4].tone_name","tot":"msg"},{"t":"set","p":"highest_tone_value","pt":"msg","to":"response.document_tone.tone_categories[0].tones[4].score","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":340,"y":500,"wires":[["86aa3f93.cd3b5","eadc9919.ca9b3"]]},{"id":"9de407a6.b8f62","type":"orion_tx","z":"ddb7a6e2.286ce","name":"send message back","orion_config":"1eb9a6af.e955e9","x":380,"y":660,"wires":[],"inputLabels":["msg.message"]},{"id":"53fe8094.52","type":"switch","z":"ddb7a6e2.286ce","name":"Ignore me","property":"sender","propertyType":"msg","rules":[{"t":"neq","v":"2462f4061dc84e97a6b99be9cd0413af","vt":"str"}],"checkall":"true","repair":false,"outputs":1,"x":460,"y":220,"wires":[["8b5da2c8.80e34"]]},{"id":"86aa3f93.cd3b5","type":"debug","z":"ddb7a6e2.286ce","name":"Debug","active":false,"tosidebar":true,"console":false,"tostatus":false,"complete":"true","x":810,"y":420,"wires":[]},{"id":"f7ac055d.fd4d48","type":"debug","z":"ddb7a6e2.286ce","name":"Debug","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload","x":810,"y":320,"wires":[]},{"id":"6fee76ee.08c0d8","type":"change","z":"ddb7a6e2.286ce","name":"Craft message","rules":[{"t":"set","p":"message","pt":"msg","to":"\"It sounds like you are experiencing\" & $.highest_tone","tot":"jsonata"}],"action":"","property":"","from":"","to":"","reg":false,"x":140,"y":640,"wires":[["9de407a6.b8f62"]]},{"id":"eadc9919.ca9b3","type":"switch","z":"ddb7a6e2.286ce","name":"ensure confidence","property":"highest_tone_value","propertyType":"msg","rules":[{"t":"gt","v":"0","vt":"num"},{"t":"else"}],"checkall":"true","repair":false,"outputs":2,"x":630,"y":500,"wires":[["6fee76ee.08c0d8","f196aedb.d3ae68"],[]]},{"id":"f196aedb.d3ae68","type":"change","z":"ddb7a6e2.286ce","name":"Craft error message","rules":[{"t":"set","p":"message","pt":"msg","to":"I'm sorry I couldn't tell how you feel.","tot":"str"}],"action":"","property":"","from":"","to":"","reg":false,"x":140,"y":680,"wires":[["9de407a6.b8f62"]]},{"id":"80966da8.ced1","type":"comment","z":"ddb7a6e2.286ce","name":"Ignore myself == no loop","info":"Because I'm sending and receiving from the same user, we'd hit an loop if I don't exclude myself.  Let's talk to our friends.","x":470,"y":180,"wires":[]},{"id":"1eb9a6af.e955e9","type":"orion_config","z":"","group":"14d2c07390f3475a910e133c3f915b21","name":""}]
```