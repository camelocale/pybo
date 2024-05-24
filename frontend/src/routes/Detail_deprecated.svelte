<script>
    import fastapi from "../lib/api"
    import Error from "../components/Error.svelte"

    export let params = {} //Detail 컴포넌트를 호출할 때 전달한 파라미터 값을 읽으려면 params 변수를 선언해야 한다. 
    // console.log(params)
    let question_id = params.question_id
    // console.log('question_id:' + question_id)

    let question = {answers:[]}
    let content = ""
    let error = {detail:[]}

    function get_question() {
        fastapi("get", "/api/question/detail/" + question_id, {}, (json => {
            question=json
        }))
    }

    get_question()

    function post_answer(event) {
        event.preventDefault() // form 태그에서 자동으로 api사 전송되는 것을 방지하기 위함이다. 
        let url = "/api/answer/create/" + question_id
        let params = {
            content: content 
        }
        fastapi('post', url, params,
            (json) => {
                content = ''
                error = {detail: []}
                get_question()
            },
            (err_json) => {
                error = err_json
            }
        )
    }

</script>

<h1>{question.subject}</h1>
<div>
    {question.content}
</div>

<ul>
    {#each question.answers as answer}
        <li>{answer.content}</li>
    {/each}
</ul>
<Error error = {error} />

<!-- 텍스트 창에서 작성한 내용이 content 변수와 연결되도록 bind:value 속성을 사용한다.  -->
<!-- 버튼을 누르면 post_answer 함수가 호출되도록 on:click 함수를 사용한다.  -->
<form method="post">
    <textarea rows="15" bind:value={content}></textarea>
    <input type="submit" value="답변등록" on:click="{post_answer}"> 
</form>

<style>
    textarea {
        width:100%;
    }
    input[type=submit] {
        margin-top:10px;
    }    
</style>


