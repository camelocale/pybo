<script>
    import fastapi from "../lib/api"
    import Error from "../components/Error.svelte"
    import { link, push } from 'svelte-spa-router'
    import { is_login, username } from  "../lib/store"

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

    function delete_question(_question_id) {
        if(window.confirm('정말로 삭제하시겠습니까?')) {
            let url = "/api/question/delete"
            let params = {
                question_id: _question_id
            }
            fastapi('delete', url, params, 
                (json) => {
                    push('/')
                },
                (err_json) => {
                    error = err_json
                }
            )
        }
    }

    function delete_answer(_answer_id) {
        if(window.confirm('정말로 삭제하시겠습니까?')) {
            let url = "/api/answer/delete"
            let params = {
                answer_id: _answer_id
            }
            fastapi('delete', url, params, 
                (json) => {
                    get_question()
                },
                (err_json) => {
                    error = err_json
                }
            )
        }
    }

</script>


<div class="container my-3">
    <!-- 질문 -->
    <h2 class="border-bottom py-2">{question.subject}</h2>
    <div class="card my-3">
        <div class="card-body">
            <div class="card-text" style="white-space: pre-line;">{question.content}</div>
            <div class="d-flex justify-content-end">
                {#if question.modify_date }
                <div class="badge bg-light text-dark p-2 text-start mx-3">
                    <div class="mb-2">modified at</div>
                    <div>{question.modify_date}</div>
                </div>
                {/if}
                <div class="badge bg-light text-dark p-2 text-start">
                    <div class="mb-2">{ question.user ? question.user.username : ""}</div>
                    <div>{question.create_date}</div>
                </div>
            </div>
            <div class="my-3">
                {#if question.user && $username === question.user.username }
                <a use:link href="/question-modify/{question.id}"
                    class="btn btn-sm btn-outline-secondary">수정</a>
                <button class="btn btn-sm btn-outline-seondary"
                    on:click={() => delete_question(question.id)}>삭제</button>
                {/if}
            </div>
        </div>
    </div>

    <button class="btn btn-secondary" on:click="{() => {push('/')}}">
        목록으로
    </button>

    <!-- 답변 목록 -->
    <h5 class="border-bottom my-3 py-2">{question.answers.length}개의 답변이 있습니다.</h5>
    {#each question.answers as answer}
    <div class="card my-3">
        <div class="card-body">
            <div class="card-text" style="white-space: pre-line;">{answer.content}</div>
            <div class="d-flex justify-content-end">
                {#if answer.modify_date }
                <div class="badge bg-light text-dark p-2 text-start mx-3">
                    <div class="mb-2">modified at</div>
                    <div>{answer.modify_date}</div>
                </div>
                {/if}
                <div class="badge bg-light text-dark p-2 text-start">
                    <div class="mb-2">{ question.user ? question.user.username : ""}</div>
                    <div>{answer.create_date}</div>
                </div>
            </div>
            <div class="my-3">
                {#if answer.user && $username === answer.user.username }
                <a use:link href="/answer-modify/{answer.id}" 
                    class="btn btn-sm btn-outline-secondary">수정</a>
                <button class="btn btn-sm btn-outline-secondary"
                    on:click={() => delete_answer(answer.id) }>삭제</button>
                {/if}
            </div>
        </div>
    </div>
    {/each}
    <!-- 답변 등록 -->
    <Error error={error} />
    <form method="post" class="my-3">
        <div class="mb-3">
            <textarea rows="10" bind:value={content} disabled={$is_login ? "": "disabled"} class="form-control" />
        </div>
        <input type="submit" value="답변등록" class="btn btn-primary {$is_login ? "": "disabled"}" on:click="{post_answer}" />
    </form>
</div>