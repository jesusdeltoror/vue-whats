<script setup lang="ts">
import QRCode from 'qrcode'
import {Buffer} from 'buffer'
import axios, { AxiosRequestConfig } from 'axios';
import FormData from 'form-data';

//INTERFACE----------------------------
interface MessageData {
    messaging_product: string;
    to: string;
    type: string;
    template: {
      name: string;
      language: {
        code: string;
      };
      components: [
        {
          type: string;
          parameters: [
            {
              type: string;
              image?: {
                id: string;
              };
              text?: string;
            }
          ]
        },
        {
          type: string;
          parameters: [
            {
              type: string;
              text: string;
            },
            {
              type: string;
              text: string;
            }
          ]
        }
      ]
    };
  }

//VARIABLES----------------------------
let botId: string = '491392060716085';
let phoneNbr: string = '52871266 9231';
let bearerToken: string = 'EAAcPz29isEIBO9ylip0jw5dk1k6XXdQ3itw1AzcQ5mkKRaaZB0bZBRU6nUBOYmHOW1nZAmlT7RU2FEAdmitD6PA0gffIOZCrZAfXf8onHhDWLdF7hJOpI01E8XsOo8tctc3HuOXvC6IAwqmcZCfLQZBElR9yjZC7R4QX9dAxsf0WdjtgpjRmvCmnYMknEgIxVdtgW6aoSFidZAkLHfYoBNJkwHSKZC';
let url: string = `https://graph.facebook.com/v15.0/${botId}/`
let textOrderService: string = '10101088';
let textOperatorName: string = 'ABDO SABAG II confirma si te llego';
//CODE QR GENERATOR--------------------------------
QRCode.toDataURL(textOrderService, {
    errorCorrectionLevel: 'Q',
  })
    .then((url: string) => {
      //document.getElementById('serviceOrderNumQR').setAttribute('src', url);
      //document.getElementById('serviceOrderNum').innerText =
        'ORDEN DE SERVICIO: 1029479'
        uploadImage(url)
    })
    .catch((err: any) => {
      console.error(err);
    });

//UPLOAD IMG API GRAPH META-----------------------    
let uploadImage = async (image64: string): Promise<void> => {
    const buffer = Buffer.from(image64.replace('data:image/png;base64,', ''), 'base64'); 
    const blob = new Blob([buffer], { type: 'image/png'});
    
   
    let formData: FormData = new FormData();
        formData.append('file', blob, 'image.png');
        formData.append('messaging_product', 'whatsapp');

    let config: AxiosRequestConfig = {
        method: 'POST',
        headers: {
            'Authorization': 'Bearer ' + bearerToken,
            'Content-Type': 'multipart/form-data',
        },
        data: formData,
};
  try {
    let response = await axios(`${url}media`, config);
    let data = response.data;
    if (data.id) {
      console.log('Image ID:', data.id);
      await sendMessage(data.id);
    } else {
      console.error('Error to upload image:', data);
    }
  } catch (error) {
    console.error('Error:', error);
  }
};

const sendMessage = async (imageId: string): Promise<void> => {
    const messageData: MessageData = {
      messaging_product: 'whatsapp',
      to: phoneNbr,
      type: 'template',
      template: {
        name: 'mclg_qr2',
        language: { code: 'es_MX' },
        components: [
          {
            type: 'header',
            parameters: [
              {
                type: 'image',
                image: {
                  id: imageId,
                },
              },
            ],
          },
          {
            type: 'body',
            parameters: [
              {
                type: 'text',
                text: textOrderService,
              },
              {
                type: 'text',
                text: textOperatorName,
              },
            ],
          },
        ],
      },
    };
    let config: AxiosRequestConfig = {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${bearerToken}`,
        'Content-Type': 'application/json',
      },
      data: JSON.stringify(messageData),
    };
  
    try {
      let response = await axios(`${url}messages`, config);
      let result = response.data;
  
      console.log('Response from sending message:', result);
    } catch (error) {
      console.error('Error sending message:', error);
    }
  };
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />
  </header>

  <main>
    
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
