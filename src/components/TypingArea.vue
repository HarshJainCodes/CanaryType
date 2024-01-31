<template>
    <h1 class="timer">
        {{ time }}
    </h1>
    <div class="textToType">
        <div class="textToType1">
            {{ testText }}
        </div>

        <div class="resultDisplay">
        </div>
        
        <textarea name="typingArea" id="" v-on:keydown="startTimer" spellcheck="false"></textarea>
    </div>

    <div class="typing-result" v-if="typingFinished">
        Your typing speed is {{ typingSpeed }} WPM
    </div>

    <div>
        <div class="afterTypingFinishedLayout">
            <div class="leftLayout">
                <label>Enter your name</label>
                <input class="typerNameInput" v-model="typerName">
                <div style="display: flex; justify-content: center;">
                    <button class="button-45" @click="saveResult" >
                        Save your result
                    </button>
                </div>
            </div>
            <div class="rightLayout">
                <h1>
                    Leaderboard
                </h1>
                <table>
                    <thead>
                        <tr>
                            <th>
                                Rank
                            </th>
                            <th>
                                Name
                            </th>
                            <th>
                                Best Typing Speed
                            </th>
                        </tr>
                    </thead>
                    <tbody>
                    </tbody>
                </table>
            </div>
        </div>
        
    </div>
    
</template>

<script>
import { ref, onMounted } from 'vue'
    export default {
        setup(){
            const typerName = ref('')
            const testText = ref('Loading data... Please wait')
            const typingStarted = ref(false)
            const time = ref(10)
            const typingFinished = ref(false)
            const typingSpeed = ref(0)
            const leaderboardFetched = ref(false)

            function calculateTypingSpeed() {
                const wordsTyped = document.querySelector('textarea').value.split(' ')
                const availablewords = testText.value.split(' ')

                let charsInTotalCorrectWords = 0;

                for (let word of wordsTyped){
                    for (let word2 of availablewords){
                        if (word === word2){
                            charsInTotalCorrectWords += word.length;
                            break
                        }
                    }
                }

                typingSpeed.value = Math.round(((charsInTotalCorrectWords) / 5) * 6)
                document.querySelector('textarea').blur();
            }

            function addSpanAroundText(){
                setTimeout(() => {
                    const enteredText = document.querySelector('textarea').value;

                    const resultDisplayElement = document.querySelector('.resultDisplay')
                    resultDisplayElement.innerHTML = ''

                    for (let i = 0; i < enteredText.length; i++){
                        if (enteredText[i] == testText.value[i]){
                            resultDisplayElement.innerHTML += `<span class="rightWord";">${enteredText[i]}</span>`
                        }else{
                            resultDisplayElement.innerHTML += `<span style="color: red;">${testText.value[i]}</span>`
                        }
                    }
                }, 10);
            }

            function startTimer(){
                if (!typingStarted.value){
                    typingStarted.value = true;
                    const timer = setInterval(() => {
                        if (time.value > 0){
                            time.value--;
                        }else{
                            typingFinished.value = true
                            calculateTypingSpeed();
                            clearInterval(timer)
                        }
                    }, 1000);
                }
                addSpanAroundText()
            }

            function getStringToTypeFromBackend(){
                // use locahost for if backend is not hosted
                fetch("https://canarytype.azurewebsites.net/api/Student/RandomWords", {
                    method: 'GET'
                }).then(response => 
                    response.json()
                ).then(data => {
                    testText.value = ''
                    for (let i = 0; i < data.length - 2; i++){
                        testText.value += data[i] += ' '
                    }
                    testText.value += data[data.length - 1]
                })
            }

            function fetchResultFromDB() {
                // fetch data from the backend
                fetch('https://canarytype.azurewebsites.net/api/Student/Leaderboard', {
                    method: 'GET'
                }).then(response => response.json())
                .then(backendData => {
                    document.querySelector('tbody').innerHTML = ''

                    for (let i = 0; i < backendData.length; i++){
                        document.querySelector('tbody').innerHTML += tableRowTemplate(i + 1, backendData[i].name, backendData[i].bestTypingSpeed)
                    }
                })
            }

                async function saveResult () {
                // first check if that name already exists in the database or not
                // fetch('https://canarytype.azurewebsites.net/api/Student/CanaryNames', {
                //     method: 'GET'
                // }).then(
                //     response => response.json()
                // ).then(data => {
                //     for (let i = 0; i < data.length; i++){
                //         if (this.typerName === data[i]){
                //             let form = new FormData()
                //             form.append("Name", this.typerName)
                //             form.append("BestTypingSpeed", this.typingSpeed)
                //             fetch('https://canarytype.azurewebsites.net/api/Student/UpdateExistingCanary', {
                //                 method: 'PUT',
                //                 body: form
                //             }).then(() => {
                //                 this.fetchResultFromDB()
                //                 this.leaderboardFetched = true
                //             }
                //             )
                            
                //             return
                //         }
                //     }
                //     // if this is a new typer, then we need to add the typer to the database
                //     let form = new FormData()
                //     form.append("Name", this.typerName)
                //     form.append("BestTypingSpeed", this.typingSpeed)
                //     fetch('https://canarytype.azurewebsites.net/api/Student/CreateNewTyper', {
                //         method: 'POST',
                //         body: form
                //     }).then(() => {
                //         this.fetchResultFromDB()
                //         this.leaderboardFetched = true
                //     })
                // })

                // lets try to rerun the same code using async await

                const canaryNames = await fetch('https://canarytype.azurewebsites.net/api/Student/CanaryNames', {
                    method: 'GET'
                })

                for (let i = 0; i < canaryNames.length; i++){
                    if (typerName.value === canaryNames[i]){
                        let form = new FormData()
                        form.append("Name", typerName.value)
                        form.append("BestTypingSpeed", typingSpeed.value)
                        await fetch('https://canarytype.azurewebsites.net/api/Student/UpdateExistingCanary', {
                            method: 'PUT',
                            body: form
                        })
                        fetchResultFromDB()
                        leaderboardFetched.value = true
                        return
                    }
                }
                let form = new FormData()
                form.append("Name", typerName.value)
                form.append("BestTypingSpeed", typingSpeed.value)
                await fetch('https://canarytype.azurewebsites.net/api/Student/CreateNewTyper', {
                    method: 'POST',
                    body: form
                })
                fetchResultFromDB()
                leaderboardFetched.value = true
            }

            function tableRowTemplate(rank, name, bestTypingSpeed){
                return `
                    <tr>
                        <td>
                            ${rank}
                        </td>
                        <td>
                            ${name}
                        </td>
                        <td>
                            ${bestTypingSpeed}
                        </td>
                    </tr>
                `
            }

            onMounted(function () {
                getStringToTypeFromBackend();
                fetchResultFromDB()
            })

            return {
                typerName : typerName, 
                testText: testText, 
                typingStarted : typingStarted, 
                time : time, 
                typingFinished : typingFinished, 
                typingSpeed : typingSpeed, 
                leaderboardFetched : leaderboardFetched,
                calculateTypingSpeed : calculateTypingSpeed,
                addSpanAroundText : addSpanAroundText,
                startTimer : startTimer,
                getStringToTypeFromBackend,
                fetchResultFromDB,
                saveResult
            }
        },
        // data () {
        //     return {
        //         typerName: '',
        //         testText: 'Loading data... Please wait',
        //         typingStarted: false,
        //         time: 10,
        //         typingFinished: false,
        //         typingSpeed: 0,
        //         leaderboardFetched: false
        //     }
        // },
        // methods: {
        //     startTimer: function(){
        //         if (!this.typingStarted){
        //             this.typingStarted = true;
        //             const timer = setInterval(() => {
        //                 if (this.time > 0){
        //                     this.time--;
        //                 }else{
        //                     this.typingFinished = true
        //                     this.calculateTypingSpeed();
        //                     clearInterval(timer)
        //                 }
        //             }, 1000);
        //         }
        //         this.addSpanAroundText()
        //     },
        //     addSpanAroundText: function (){
        //         setTimeout(() => {
        //             const enteredText = document.querySelector('textarea').value;

        //             const resultDisplayElement = document.querySelector('.resultDisplay')
        //             resultDisplayElement.innerHTML = ''

        //             for (let i = 0; i < enteredText.length; i++){
        //                 if (enteredText[i] == this.testText[i]){
        //                     resultDisplayElement.innerHTML += `<span class="rightWord";">${enteredText[i]}</span>`
        //                 }else{
        //                     resultDisplayElement.innerHTML += `<span style="color: red;">${this.testText[i]}</span>`
        //                 }
        //             }
        //         }, 10);
        //     },
        //     calculateTypingSpeed: function() {
        //         const wordsTyped = document.querySelector('textarea').value.split(' ')
        //         const availablewords = this.testText.split(' ')

        //         let charsInTotalCorrectWords = 0;

        //         for (let word of wordsTyped){
        //             for (let word2 of availablewords){
        //                 if (word === word2){
        //                     charsInTotalCorrectWords += word.length;
        //                     break
        //                 }
        //             }
        //         }

        //         this.typingSpeed = Math.round(((charsInTotalCorrectWords) / 5) * 6)
        //         document.querySelector('textarea').blur();
        //     },
        //     getStringToTypeFromBackend: function(){
        //         // use locahost for if backend is not hosted
        //         fetch("https://canarytype.azurewebsites.net/api/Student/RandomWords", {
        //             method: 'GET'
        //         }).then(response => 
        //             response.json()
        //         ).then(data => {
        //             this.testText = ''
        //             for (let i = 0; i < data.length - 2; i++){
        //                 this.testText += data[i] += ' '
        //             }
        //             this.testText += data[data.length - 1]
        //         })
        //     },
        //     saveResult : async function () {
        //         // first check if that name already exists in the database or not
        //         // fetch('https://canarytype.azurewebsites.net/api/Student/CanaryNames', {
        //         //     method: 'GET'
        //         // }).then(
        //         //     response => response.json()
        //         // ).then(data => {
        //         //     for (let i = 0; i < data.length; i++){
        //         //         if (this.typerName === data[i]){
        //         //             let form = new FormData()
        //         //             form.append("Name", this.typerName)
        //         //             form.append("BestTypingSpeed", this.typingSpeed)
        //         //             fetch('https://canarytype.azurewebsites.net/api/Student/UpdateExistingCanary', {
        //         //                 method: 'PUT',
        //         //                 body: form
        //         //             }).then(() => {
        //         //                 this.fetchResultFromDB()
        //         //                 this.leaderboardFetched = true
        //         //             }
        //         //             )
                            
        //         //             return
        //         //         }
        //         //     }
        //         //     // if this is a new typer, then we need to add the typer to the database
        //         //     let form = new FormData()
        //         //     form.append("Name", this.typerName)
        //         //     form.append("BestTypingSpeed", this.typingSpeed)
        //         //     fetch('https://canarytype.azurewebsites.net/api/Student/CreateNewTyper', {
        //         //         method: 'POST',
        //         //         body: form
        //         //     }).then(() => {
        //         //         this.fetchResultFromDB()
        //         //         this.leaderboardFetched = true
        //         //     })
        //         // })

        //         // lets try to rerun the same code using async await

        //         const canaryNames = await fetch('https://canarytype.azurewebsites.net/api/Student/CanaryNames', {
        //             method: 'GET'
        //         })

        //         for (let i = 0; i < canaryNames.length; i++){
        //             if (this.typerName === canaryNames[i]){
        //                 let form = new FormData()
        //                 form.append("Name", this.typerName)
        //                 form.append("BestTypingSpeed", this.typingSpeed)
        //                 await fetch('https://canarytype.azurewebsites.net/api/Student/UpdateExistingCanary', {
        //                     method: 'PUT',
        //                     body: form
        //                 })
        //                 this.fetchResultFromDB()
        //                 this.leaderboardFetched = true
        //                 return
        //             }
        //         }
        //         let form = new FormData()
        //         form.append("Name", this.typerName)
        //         form.append("BestTypingSpeed", this.typingSpeed)
        //         await fetch('https://canarytype.azurewebsites.net/api/Student/CreateNewTyper', {
        //             method: 'POST',
        //             body: form
        //         })
        //         this.fetchResultFromDB()
        //         this.leaderboardFetched = true

        //     },
        //     tableRowTemplate: function (rank, name, bestTypingSpeed){
        //         return `
        //             <tr>
        //                 <td>
        //                     ${rank}
        //                 </td>
        //                 <td>
        //                     ${name}
        //                 </td>
        //                 <td>
        //                     ${bestTypingSpeed}
        //                 </td>
        //             </tr>
        //         `
        //     }
        //     ,
        //     fetchResultFromDB: function() {
        //         // fetch data from the backend
        //         fetch('https://canarytype.azurewebsites.net/api/Student/Leaderboard', {
        //             method: 'GET'
        //         }).then(response => response.json())
        //         .then(backendData => {
        //             document.querySelector('tbody').innerHTML = ''

        //             for (let i = 0; i < backendData.length; i++){
        //                 document.querySelector('tbody').innerHTML += this.tableRowTemplate(i + 1, backendData[i].name, backendData[i].bestTypingSpeed)
        //             }
        //         })
        //     }
        // },
        // mounted: function(){
        //     this.getStringToTypeFromBackend();
        //     this.fetchResultFromDB()
        // }
    }
</script>

<style>
header {
    font-family: sans-serif;
    font-size: 40px;
}

.rightWord {
    color: rgb(61, 255, 61);
}

.timer {
    width: 80vw;
    margin-left: auto;
    margin-right: auto;
    text-align: left;
    margin-top: 200px;
    margin-bottom: 0px;
}

.textToType {
    /* position: absolute;
    width: 80vw; */
    display: grid;

    margin-top: 5px;
    font-size: 30px;
    margin-left: auto;
    margin-right: auto;
    width: 80vw;
    text-align: left;
}

.textToType1 {
    grid-column: 1;
    grid-row: 1;

    opacity: 50%;

    font-family: 'Roboto Mono', monospace;
}


.resultDisplay {
    grid-column: 1;
    grid-row: 1;

    width: 80vw;
    margin-left: auto;
    margin-right: auto;

    font-family: 'Roboto Mono', monospace;
}


textarea {
    grid-column: 1;
    grid-row: 1;
    color: transparent;
    caret-color: black;

    width: 80vw;
    font-size: 30px;

    font-family: 'Roboto Mono', monospace;

    border: none;
    padding: 0;
    white-space: normal;
    text-align: left;
    background: transparent;
    z-index: 5;
}

textarea:focus-visible {
    outline: none;
}

.typing-result {
    font-family: 'Roboto Mono', monospace;
    
    margin: 30px;
    font-size: 30px;
}

label {
    font-size: 25px;
}

.typerNameInput{
    margin-top: 30px;
    margin-left: 20px;
    margin-right: 20px;
    font-size: 30px;
    color: green;
}

.saveResult {
    margin-top: 30px;
    font-size: 20px;
    padding: 10px;
}

.afterTypingFinishedLayout {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 10px;
}

.leftLayout{
    grid-column: 1;
    grid-row: 1;
}

.rightLayout{
    display: flex;
    justify-content: center;
    flex-direction: column;

    grid-column: 2;
    grid-row: 1;
    justify-content: center;
}

table {
    border-collapse: collapse;
    margin: 25px 0;
    font-size: 0.9em;
    font-family: sans-serif;
    min-width: 400px;
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.15);
}

thead tr {
    background-color: #009879;
    color: #ffffff;
    text-align: center;
}

th, td {
    padding: 12px 15px;
}

tbody tr {
    border-bottom: 1px solid #dddddd;
}

tbody tr:nth-of-type(even) {
    background-color: #f3f3f3;
}

tbody tr:last-of-type {
    border-bottom: 2px solid #009879;
}

tbody tr.active-row {
    font-weight: bold;
    color: #009879;
}

tbody tr:hover {
    background-color: #a2d9ce;
}

/* <!-- HTML !-->
<button class="button-45" role="button">Button 45</button> */

/* CSS */
.button-45 {
  /* align-items: center; */
  background-color: #87fba4;
  /* background-position: 0 0; */
  padding: 10px;
  margin-top: 60px;
  border: 1px solid #004d20;
  border-radius: 11px;
  box-sizing: border-box;
  color: #013729;
  cursor: pointer;
  display: flex;
  font-size: 1rem;
  font-weight: 700;
  line-height: 33.4929px;
  list-style: outside url(https://www.smashingmagazine.com/images/bullet.svg) none;
  padding: 2px 12px;
  text-align: left;
  text-decoration: none;
  text-shadow: none;
  text-underline-offset: 1px;
  transition: border .2s ease-in-out,box-shadow .2s ease-in-out;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  white-space: nowrap;
  word-break: break-word;
}

.button-45:active,
.button-45:hover,
.button-45:focus {
  outline: 0;
}


.button-45:active {
  background-color: #8ffaaa;
  box-shadow: rgba(0, 0, 0, 0.12) 0 1px 3px 0 inset;
  color: #FFFFFF;
}

.button-45:hover {
  background-color: #8ffaaa;
  border-color: #02af42;
}

.button-45:active:hover,
.button-45:focus:hover,
.button-45:focus {
  background-color: #003816;
  box-shadow: rgba(0, 0, 0, 0.12) 0 1px 3px 0 inset;
  color: #FFFFFF;
}

</style>

