# AssistenteDelivery_AWS
Criando um Assistente de Delivery com AWS Step Functions e Bedrock

# O que foi feito

Para testar um método de automação de sequências de prompts.

Foi feito o seguinte exercício de um assistente que baseado na comida desejada e cidade, recomenda combinações de comida, bebidas e locais para a refeição.

# Código do Asistente de Delivery

"""
{
  "Comment": "An example of using Bedrock to chain prompts and their responses together.",
  "StartAt": "Pergunta combinações",
  "States": {
    "Pergunta combinações": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Parameters": {
        "ModelId": "arn:aws:bedrock:us-east-2::foundation-model/amazon.titan-embed-text-v2:0",
        "Body": {
          "inputText": "Estou planejando um jantar romântico, e vou pedir macarrão. Liste dois itens que combinam para uma experiência gastronômica",
          "textGenerationConfig": {
            "temperature": 0,
            "topP": 1,
            "maxTokenCount": 512
          }
        },
        "ContentType": "application/json",
        "Accept": "*/*"
      },
      "Next": "Add first result to conversation history",
      "ResultPath": "$.result_one",
      "ResultSelector": {
        "result_one.$": "$.Body.inputText[0].text"
      }
    },
    "Add first result to conversation history": {
      "Type": "Pass",
      "Next": "Pergunta bebida",
      "Parameters": {
        "convo_one.$": "States.Format('{}\n{}', $.prompt_one, $.result_one.result_one)"
      },
      "ResultPath": "$.convo_one"
    },
    "Pergunta bebida": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Parameters": {
        "ModelId": "arn:aws:bedrock:us-east-2::foundation-model/amazon.titan-embed-text-v2:0",
        "Body": {
          "inputText": "Liste três bebidas para acompanhar o jantar",
          "textGenerationConfig": {
            "temperature": 0,
            "topP": 1,
            "maxTokenCount": 512
          }
        },
        "ContentType": "application/json",
        "Accept": "*/*"
      },
      "Next": "Add second result to conversation history",
      "ResultSelector": {
        "result_two.$": "$.Body.inputText[0].text"
      },
      "ResultPath": "$.result_two"
    },
    "Add second result to conversation history": {
      "Type": "Pass",
      "Next": "Pergunta local",
      "Parameters": {
        "convo_two.$": "States.Format('{}\n{}\n{}', $.convo_one.convo_one, $.prompt_two, $.result_two.result_two)"
      },
      "ResultPath": "$.convo_two"
    },
    "Pergunta local": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Parameters": {
        "ModelId": "arn:aws:bedrock:us-east-2::foundation-model/amazon.titan-embed-text-v2:0",
        "Body": {
          "inputText": "Recomende um local para ter um jantar romântico em São Paulo",
          "textGenerationConfig": {
            "temperature": 0,
            "topP": 1,
            "maxTokenCount": 512
          }
        },
        "ContentType": "application/json",
        "Accept": "*/*"
      },
      "End": true,
      "ResultSelector": {
        "result_three.$": "$.Body.inputText[0].text"
      }
    }
  }
}

"""
