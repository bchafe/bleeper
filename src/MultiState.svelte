<script>
    import { onMount } from 'svelte'
    
        export let key = [];
        export let value = [];
        
        let length = key.length;
        let currentKey = key[0];
        export let currentValue = value[0];
        export let defaultIndex = 0;
        let currentIndex = 0;
        let padding = 0;
    
    
        
        onMount(() => {
            currentIndex = defaultIndex;
            handleChange();
        })
    
        let handleChange = () => {
            currentKey = key[currentIndex];
            currentValue = value[currentIndex];
        }
    
        let handleForward = () => {
            if(currentIndex >= length-1) currentIndex = 0;
            else currentIndex++;
            handleChange();
        }
    
        let handleBack = () => {
            if(currentIndex <= 0) currentIndex = length-1;
            else currentIndex--;
            handleChange();
        }
    
    
    </script>
    <div class="container">
    <button on:click={handleBack}>&lt;</button>
    {currentKey.padStart((padding-currentKey.length)/2 + currentKey.length, ' ').padEnd(padding, ' ')}
    <button on:click={handleForward}>&gt;</button>
    </div>
    <style>
        .container{
            width: var(--width);
            display: flex;
            align-items: center;
            justify-content: space-between;
            background-color: black;
        }

        button{
            height: 40px;
            line-height: 0;
            border-radius: 0;
            background-color: black;
        }
        button:focus{
            outline: none;
        }

        button:hover{
            border-color: white;
        }
    </style>