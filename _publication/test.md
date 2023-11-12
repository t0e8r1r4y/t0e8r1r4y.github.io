<pre>
<code class="java">
public static void main() {
	return;
}
</code>
</pre>
<br>

<br> 
회사에서 애자일로 업무를 처리하다보니, 프론트엔드 개발도 찔끔하게 됨. 백엔드의 스페셜리티를 가져가야되지만… 업무상 프론트해야되면 잘 해야되니까 틈틈히 공부를 하게되었고 React로 FrontEnd에 입문하면서 찾아본 1주일간의 지식을 정리하고자 함.
<br>

<br> 
- [환경 구축] yarn, node, npm
<br> 
- [언어] JavaScript &  - TypeScript
<br> 
- [기술스택]  react, redux, recoil
<br> 
- [배포]  aws amplify ( 회사에서 사용하는 기술 스택은 아닌데, 공부하면서 배포해보기에 적합하여 선정 )
<br> 
- ui 관련 도구들
<br> 
- MDN :  - https://developer.mozilla.org/ko/ -  ( 여기를 잘 참고하여 개발한다 )
<br> 
- 디자인 도구 : Figma ( 이걸 연동해서 여기서 디자인한 내용을 그냥 반영하는 방법도 있다고 하는데…? real )
<br> 
---
<br> 
<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/728ed11b-a647-4f0b-a7f1-61c4fc89e594/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20231111%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20231111T182043Z&X-Amz-Expires=3600&X-Amz-Signature=511872dd821940e2622ea05753e23cbd4e82bc9f4532a33d005b878941ce1204&X-Amz-SignedHeaders=host&x-id=GetObject" />
<br>

<br> 
웹페이지가 렌더링 되는 과정
<br>

<br> 
1. HTML parser가 HTML을 바탕으로 DOM tree를 그린다.
<br>

<br> 
2. CSS parser가 CSS를 바탕으로 CSSOM을 그린다.
<br>

<br> 
3. DOM에 CSSOM을 적용하여 Render Tree를 그린다.
<br>

<br> 
4. Render Tree를 바탕으로 Painting 하여 실제 화면에 렌더링 한다.
<br>

<br> 
- HTML 코드를 읽어 내려가다가 <script></script> 태그를 만나면 파싱을 잠시 중단하고 js 파일을 로드한다.
<br> 
DOM : 웹 브라우저와 JS가 Html을 파싱하여 트리구조로 파싱하여 만든 객체
<br>

<br> 
CSSOM(Cascading Style Sheets Object Model) : DOM + CSS
<br>

<br> 
초기 웹이 나왔을 때는 정적 웹으로 위 내용으로 화면을 구성했음. 하지만 사용자와의 interaction이 중요해지면서 웹을 동적으로 제어해야되는 이슈가 생기게 됨. DOM을 동적으로 제어하기 위해 DOM API가 생겼다. 이를 통해서 JS를 사용하여 DOM에 요소를 추가,수정,삭제하는 행위를 통해서 동적으로 DOM을 제어했다. 
<br>

<br> 
<div class="highlight"> DOM에 대한 상세 설명은 MDN 참고 :  https://developer.mozilla.org/ko/docs/Web/API/Document_Object_Model/Introduction </div>
<br>

<br> 
DOM을 동적으로 제어해야하는 사용자와의 Interaction이 많아지면서 브라우저에서 UI 변경과 데이터 호출 등에 많은 연산이 발생하게 되어 브라우저에서의 성능이슈가 발생하는 부분이 생겼다. 기존의 jquery같은 기술로는 DOM을 직접 동적으로 제어하려고 하기 때문에 그렇다고한다. 그래서 Virtual DOM이라는 컨셉이 등장하게 된다.
<br>

<br> 
<div class="highlight"> Virtual DOM에 대한 내용은 React Docs 참고 :  https://legacy.reactjs.org/docs/faq-internals.html </div>
<br>

<br> 
Virtual DOM 컨셉으로 웹을 동적으로 조작하는 행위는 아래와 같다고 함.
<br>

<br> 
<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/256cbb7d-0abd-4470-ab50-599554be8f8c/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20231111%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20231111T182043Z&X-Amz-Expires=3600&X-Amz-Signature=7ae267470d595805cbaaa54cc805fab6aa1dcfa7f6668067e4a701b593895276&X-Amz-SignedHeaders=host&x-id=GetObject" />
<br>

<br> 
Diffing 알고리즘 :  https://ko.legacy.reactjs.org/docs/reconciliation.html
<br>

<br> 
<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/7b7a4820-ad59-4a7d-8c28-b6de9274c55e/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20231111%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20231111T182043Z&X-Amz-Expires=3600&X-Amz-Signature=3541681f87f1136cfaa254e987d1e354a961a8caf0d56d1886dd1094df054cec&X-Amz-SignedHeaders=host&x-id=GetObject" />
<br>

<br> 
위 내용이 React에서 Rerendering하는 과정(?)이라고 대충 이해했다. React는 상태나 속성 값이 변경되게 되면 리렌더링이라는 과정을 통해서 화면 값을 갱신하게 된다. 그 과정에서 빠르게 값을 변경 시킬 수 있는 것이 Virtual DOM 컨셉을 가지고 변경 부분만 다시 뿌리기 때문에 빠르고, 사용자가 Real DOM을 직접 제어하는 것보다 안전하다고 한다.
<br>

<br> 
간략하게 정리한 부분이라 깊이있게 다루지는 못했지만 CSR(Client Side Render), SSR(Server Side Render) 등의 개념도 함께 알아야 한다. 결국 아래 그림처럼 client에서 서버에 뭘 요청하고 서버가 뭘 주는지 그리고 그게 위 내용와 물려서 어떻게 동작되는지는 알아야 한다. 이런 개념은 드림코딩이라는 유툽 채널에서 개념 정리를 잘 해주는 듯 하다. (  https://youtu.be/iZ9csAfU5Os  )
<br>

<br> 
<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/340cdaa6-c689-4e3c-9bc0-72bfbf95122f/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20231111%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20231111T182043Z&X-Amz-Expires=3600&X-Amz-Signature=8dbf2a8c2d1e585a551e8ae228c6fcd69a8558840b6cbdd0ed2cd6bb6c4c54cb&X-Amz-SignedHeaders=host&x-id=GetObject" />
<br>

<br> 
그래서 React에 대해서 요약하면 아래와 같다.
<br>

<br> 
- 컴포넌트의 상태값이 변경될 때마다 UI를 자동으로 업데이트해주는 JS 라이브러리다.
<br> 
- Virtual DOM을 통해 변경된 부분만 효율적으로 업데이트를 해주는 구조다.
<br> 
---
<br> 
대게 많은 예제들은 React App만 만들어서  localhost:3000  띄우고 짜짱~! 하고 끝이난다. 근데 현업에서 업무를 하면서 느끼는 점은 배포까지는 한 묶음이다. 개발만 실컷하고 정작 배포가 안된다면 말짱 꽝이라는 생각에, React App의 생성과 배포를 한 사이클에 묶어서 시작해본다.
<br>

<br> 
회사에서는 CloudFront와 Docker Image로 묶어서 뭔가 배포를한다고 들었는데, 나는 빠른 배포와 백엔드 API를 별도 구축하지 않고 Front만 빠르게 개발을 진행하여 보고자 Aws Amplify를 선택했다. 빠른 개발 및 테스트를 위해서 노마드 코더의 예제를 먼저 띄워볼것이다. (  https://www.youtube.com/watch?v=o7FkmtqIYOE  )
<br>

<br> 
그리고 위 예제를 선택한 이유는 GraphQL을 사용하고 있기 때문이기도하다. GraphQL은 Tweeter나 Insta와 같이 뭔가 사용자 인터랙션이 많고 데이터 변경이 잦은 서비스들에서 많이 사용한다고 한다. 그리고 학습 목적으로 이것저것 API를 추가하다보면, 필요한것만 호출하고 싶을 것 같아서 GraphQL을 사용하는 것을 좀 선호한다.
<br>

<br> 
<div class="highlight"> AWS Amplifier에 대한 전반적인 내용은 Dev Center 문서를 볼 것 :  https://docs.amplify.aws/lib/graphqlapi/authz/q/platform/ios/#api-key </div>
<br>

<br> 
Amplify 그래서 넌 뭐니?
<br>

<br> 
- Front Hosting을 해주는 도구 + 백엔드 환경도 대충 만들어 줌
<br> 
설치
<br>

<br> 
설치는 front, back 두 스텝으로 구분이 된다. amplify cli를 사용해서 backend api를 먼저 생성하고, front는 배포만 진행하면 된다.
<br>

<br> 
back
<br>

<br> 
- 아래 링크를 쭉 따라가면 될 듯 (  - https://docs.amplify.aws/cli/start/install/ -  ) 혹은 내가 진행한건 아래와 같음
<br> 
- 내가 설치한 것 ( 백엔드쪽 환경 구성)
<br> 
<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/55dd362f-5d20-4ab7-86a3-89fbdf834f83/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20231111%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20231111T182043Z&X-Amz-Expires=3600&X-Amz-Signature=c71029b3bbd085f60ddbfd8ac493cadcdd9eba1955032df833494719348bbefa&X-Amz-SignedHeaders=host&x-id=GetObject" />
<br>

<br> 
요렇게 되고 Amplify Studio에서 편집도 가능하다. 그러나 아래 처럼 IDE에서 편집해서 amplify push 해서 반영하는게 편리한 것 같다.
<br>

<br> 
<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/5e5c788b-11b7-4971-bdb8-de720cf44558/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20231111%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20231111T182043Z&X-Amz-Expires=3600&X-Amz-Signature=7ceb447486636ca7e379934115d79963d6201f0c4b239932c91a34ca12bd90c0&X-Amz-SignedHeaders=host&x-id=GetObject" />
<br>

<br> 
front 
<br>

<br> 
<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/b97051d4-48b5-472c-8b40-37f8c75342e7/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20231111%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20231111T182043Z&X-Amz-Expires=3600&X-Amz-Signature=0b079e89b1fc2265d92b1bdbe6299466651642f7598fb94e361b09bbc54f569c&X-Amz-SignedHeaders=host&x-id=GetObject" />
<br>

<br> 
- 배포는 수동 방식과 자동 방식이 있다.
<br> 
- 나는 자동 방식으로 진행했고 git의 특정 branch에 올리면 배포가 진행되는 방식을 사용했다. git 인증을 통해서 연동을 해두고 특정 branch에 push 되면 배포가 진행된다.
<br> 
- 빌드 설정을 보면 결국 build할 이미지에 front app을 띄워주는 형태라는 것을 알수 있다. 그리고 그 이미지는 설정을 할 수 있다.
<br> 
- 그리고 환경변수로 포함되는 값들을 추가할 수도 있다.
<br> 
- 그리고 cloud watch가 연동되어 있어 모니터링도 가능하다.
<br> 
- 그리고 엑세스 제어도 가능하다. 일단 지금 배포해둔 앱은 간단하게 특정 유저 정보를 입력하면 액세스가 가능한 형태인데, congnito 등을 연동해서 로그인과 인증을 구현할 수도 있다.
<br> 
front와 back이 별도로 이루어진다. 그리고 배포시에 sync가 맞지 않으면 배포에 실패하므로, 환경변수 세팅 등에 유의해야 한다. 하지만, 기본 튜토리얼만 쭉 따라가도 배포가 가능하고 CI/CD도 별도로 구축할 것 없이 github을 통해서 간단하게 처리할 수 있다. 개발에 집중 할 수 있게 만들어주는 아주 훌륭한 도구라고 생각한다.
<br>

<br> 
이렇게까지하면 그냥 앱만 뚝 만들어서 로컬에 짜장~!하는 것 보다 아래 장점이 있는 것 같다.
<br>

<br> 
- project 브랜치 관리
<br> 
- 배포 경험 + 배포 환경에 따른 ui 편집
<br> 
- 사용자 테스트
<br> 
실제 생성 완료한 앱은 아래와 같다. 이제 이걸 수정하면서 react 찍먹에 들어간다.
<br>

<br> 
<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/e8d1ca10-2b95-42f9-a80d-b65b84ed43fb/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20231111%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20231111T182043Z&X-Amz-Expires=3600&X-Amz-Signature=973b7e82a233eefb410a9329bc6996ca79ff8c61688dfc790ea88a1225c4e45f&X-Amz-SignedHeaders=host&x-id=GetObject" />
<br>

<br>
