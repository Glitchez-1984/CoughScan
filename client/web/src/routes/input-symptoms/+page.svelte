<script lang="ts">
    import { symptomInputOptionLabels } from "../../lib/symptomInputOptions";
    import type { PageData } from "./$types";

    export function handleCheck(e: Event) {
        (document.getElementsByClassName((e.target as HTMLInputElement).dataset["name"]!)[0] as HTMLInputElement).value = (e.target as HTMLInputElement).checked.toString()
    }

    async function handleSubmit(event: SubmitEvent) {
        event.preventDefault();

        const formData = new FormData(event.target as HTMLFormElement);
        const formDataObject: any = {};
        formData.forEach((value, key) => {
            formDataObject[key] = value;
        });

        let request: any = {};
        Object.keys(formDataObject).forEach(i => {
            const val = formDataObject[i];
            if (val == "true") {
                request[i] = 1;
            }
            else if (val == "false") {
                request[i] = 0;
            }
            else {
                request[i] = val;
            }
        });

        try {
            triggerLoadingScreen()
            const response = await fetch('https://coughscan.net/api/predict_symptoms', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Accept': 'application/json',
                },
                body: JSON.stringify(request)
            });

            if (response.ok) {
                const responseData = await response.json();
                console.log('Response from server:', responseData);
                const pageData: PageData = {
                    diagnosisInfo: {...responseData, type: "symptoms"},
                    symptomInputOptions: formDataObject
                };
                window.location.href = `/diagnosis/${encodeURIComponent(JSON.stringify(pageData))}`

            } else {
                console.error('Server returned an error:', response.status);
            }
        } catch (error) {
            console.error('Error while sending the data', error);
        }
    }

    function triggerLoadingScreen() {
        (document.getElementById("loading") as HTMLDivElement).style.opacity = "1";
        (document.getElementById("loading") as HTMLDivElement).style.display = "block";
        (document.getElementById("test-content") as HTMLDivElement).style.display = "none";
    }
</script>

<div id="loading">
    <div></div>
    <span>If it takes longer than 10 seconds, please refresh the page and take the test again.</span>
</div>
<link href="/css/input-symptoms.css" rel="stylesheet"/>
<main id="test-content">
    <h1>Input Symptoms</h1>
    <form action="/" method="post" on:submit={handleSubmit}>
        {#each symptomInputOptionLabels as [symptomInputOptionLabel, propertyName]}
        <div>
            <label for={propertyName}>{symptomInputOptionLabel}</label>
            <div>
                {#if symptomInputOptionLabel.includes("[")}
                <select name={propertyName}>
                    {#each symptomInputOptionLabel.substring(
                        symptomInputOptionLabel.indexOf("[") + 1,
                        symptomInputOptionLabel.indexOf("]")
                    ).split(", ") as option}
                        <option value={option.replaceAll(" ", "-").replaceAll(`'`, "").toLowerCase()}>{option}</option>
                    {/each}
                </select>
                {:else}
                <input type="checkbox" data-name={propertyName} on:change={handleCheck}/>
                <input type="hidden" name={propertyName} class={propertyName} value="false"/>
                {/if}
                <span class="style"/>
            </div>
        </div>
        {/each}
        <button type="submit">Submit</button>
    </form>
</main>