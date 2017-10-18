[ASK Documentation](https://developer.amazon.com/docs/ask-overviews/build-skills-with-the-alexa-skills-kit.html)
### Alexa Skills Kit
> Alex에게 새로운 기술을 주입할 수 있는 API   
>```When a user speaks to a device with Alexa, the speech is streamed to the Alexa service in the cloud. Alexa recognizes the speech, determines what the user wants, and then sends a structured request to the particular skill that can fulfill the user’s request.```

#### User Interaction Flow
![이미지 이름]()

마이크로 서비스 아키텍처
모놀리식 아키텍처

#### Skill Type
1.Custom Skill   

사용자가 원하는 여러 요청들을 처리하는 Skill을 직접 만들 수 있음.   
음성을 통해 사용자가 원하는 결과를 응답받기 위한 요청을 하게 된다.   
즉, 기기로 전달된 사용자의 음성에는 사용자의 `요구사항`이 있는데 이를 Intent라 한다.   

Custom Skill을 이용해 새로운 Skill을 만들기 위해서는 사용자의 어떠한 `Intent`에 대한 Skill을 만들 것 인지를 정해야한다.   
예를 들어, 날씨에 대한 정보를 알려주는 Skill을 주입하고자 한다면 Intent는 날씨가 될 것이다.    

하지만 사용자는 단순히 "날씨"라고 말하지 않고 "오늘 날씨 어때?", "오늘 날씨 알려줘" "오늘 비와?"처럼 여러가지의 문장으로 요청을 할 것이다.    
이렇게 특정 Intent를 가지고 있는 여러 문장들을 `interaction model`이라고 한다.   
하지만 같은 intent를 가지고 있는 문장들이라고 할지라도 수천,수만가지의 문장이 만들어 질 수 있다.   
그러한 문장 하나하를 interaction model로 정의하는 것은 현실적으로 불가능하다고 할 수 있다.    

이를 해결하기 위한 방법으로는 사용자의 요청을 처리해주는 가상의 주체를 만들고 이 주체에게 이름을 부여하는 방법을 사용할 수 있다.   
예를 들어 `날씨`라는 `intent`를 처리하는 `skill`을 개발하였다고 생각해보자.   
그리고 이 `skill`에 대한 가상의 이름을 `기상캐스터`라고 지었다.   
이런 경우, 사용자는 alexa에게 다음과 같이 요청을 할 수 있다.    

"`알렉사`야, `기상캐스터`한테 오늘 날씨좀 물어봐줘."   
alexa는 `기상캐스터`라는 `invocation name`을 인식하고 사용자의 요청이 `날씨` intent에 해당한다는 것을 인지하고 날씨 intent에게 요청을 보낸다.   

2.Smart Home Skill API   
집안에 있는 여러 기기들, 전등, 도어락, 보일러,TV 등을 제어할 수 잇는 Skill을 만들 수 있음.   
1번의 Custom Skill과 달리 특정 Device에 직접적으로 연동하기 위해사용하므로 비교적 쉽다.   

3.Entertainment Device Control in  the Smart Home Skill API   
2번의 Smart Home Skill API를 기반으로 Smart TV에 관련된 Skill을 만들 수 있는데 특화된 API   

4.Video Skill API   
Smart Home Skill API와 유사하지만 Video 장치에 특화된 API로 사용자의 요구에 맞는 영상물을 찾는 등의 고급 Skill을 만들 수 있음.   

5.Flash Briefing Skill API   
다른 서비스에서 제공하고 있는 음성 contents를 들려줄 수 있는 Skill을 만들 수 있음.   

#### Interaction model
>"In the context of Alexa, an interaction model is somewhat analogous to a graphical user interface in a traditional app."   
> 전통적인 GUI기반의 사용자 Interaction 처럼 음성을 통해 사용자와 Interaction 하기위한 model

intent와 sample utterances, dialog model 등으로 구성된다.   
intent는 `slots`이라는 arguments를 가질 수 있다.   
sample utterances 는 intent와 실제 사용자로 부터 얻은 단어들을 mapping하기 위해 사용되는 문장들이다.   
이러한 Interaction Model을 정의하기 위해 JSON을 사용할 수 있다.   

[intent schema in JSON foramt](https://developer.amazon.com/docs/custom-skills/define-the-interaction-model-in-json-and-text.html)   

Interaction Model은 크게 4가지의 action으로 나눌 수 있다.
1. Make a request
2. Collect more information
3. Provice needed information
4. User\`s request is completed

1.Make a request   
사용자가 취하는 action으로 Alexa에게 요청을 시작하는 단계이다.   
Alexa는 Interaction Model을 사용해 사용자의 요청을 해석하고 이를 처리할 수 있는 Skill을 찾는다.   

2.Collect more information   
최초로 사용자로부터 받은 요청에 추가적인 정보가 필요로할경우 사용되는 action이다.   
구체적인 추가정보를 사용자에게 물어본다.   

3.Provide needed information   
2번 Action을 통해 사용자는 다시 추가적인 정보를 전송한다.   

4.User\`s request is completed
사용자로부터 받은 정보들을 통해 처리가 완료된 경우 처리결과를 다시 사용자에게 보내준다.   
