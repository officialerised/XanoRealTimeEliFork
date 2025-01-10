function(instance, context) {
   const i = instance,
         c = context;
   let data = i.data,
       publish = i.publishState,
       trigger = i.triggerEvent;
   data.messages = [];
   data.payloads = [];
   xanoRealtimeListeners.push(instance);
   
   data.message_received = function(message) {
       // Special handling for system messages - store in payloads but not messages
       if (message.action === "presence_full" || message.action === "connection_status") {
           let payload = message.presence || message.status || message;
           if (typeof payload === 'object') {
               payload = JSON.stringify(payload);
           }
           
           let history_payloads = data.payloads;
           history_payloads.unshift(payload);
           data.payloads = history_payloads.slice(0, data.history_retention);
           publish('payloads', data.payloads);
           return;
       }
       
       // Process messages that contain actual records
       if (message.action === "event" && message.payload && message.payload.data) {
           try {
               // Format data to prefix _api_c2_
               const transformedData = message.payload.data.map((item) => {
                   const transformedItem = {};
                   Object.keys(item).forEach(key => {
                       transformedItem[`_api_c2_${key}`] = item[key];
                   });
                   return transformedItem;
               });
               
               let history_messages = data.messages;
               history_messages.unshift(transformedData);
               data.messages = history_messages.slice(0, data.history_retention);
               publish('messages', transformedData);
               
               // Maintain raw data in payloads
               let history_payloads = data.payloads;
               history_payloads.unshift(JSON.stringify(message.payload.data));
               data.payloads = history_payloads.slice(0, data.history_retention);
               publish('payloads', data.payloads);
               
           } catch (error) {
               console.error("Error processing message:", error);
           }
       } else {
           // Handle any other message types by storing in payloads only
           let payload = message.presence || message.status || message;
           if (typeof payload === 'object') {
               payload = JSON.stringify(payload);
           }
           
           let history_payloads = data.payloads;
           history_payloads.unshift(payload);
           data.payloads = history_payloads.slice(0, data.history_retention);
           publish('payloads', data.payloads);
       }
       
       trigger('message_received');
   }
}
