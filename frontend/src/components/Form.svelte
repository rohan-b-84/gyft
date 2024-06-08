<script lang="ts">
  import type { kFormData } from "$lib/_types";
  import { quintIn } from "svelte/easing";
  import { fade } from "svelte/transition";
  import Header from "./Header.svelte";
  import Footer from "./Footer.svelte";
  import { writable } from "svelte/store";

  let securityQuestion = writable("");
  let fetchedSecurityQuestion = writable(false);
  let otpSent = writable(false);

  export let formData: kFormData;

  function getFormData() {
    const _formData = new FormData();
    _formData.append("roll", formData.roll || "");
    _formData.append("pass", formData.pass || "");
    _formData.append("secret_answer", formData.secret_answer || "");
    _formData.append("email_otp", formData.email_otp || "");
    return _formData;
  }

  async function getSecurityQuestion() {
    console.log(formData.roll);
    try {
      const res = await fetch("http://127.0.0.1:5000/secret-question", {
        method: "POST",
        body: getFormData(),
      });

      if (!res.ok) {
        throw new Error("Network response was not ok");
      }

      const data = await res.text();
      securityQuestion.set(data); // Update the securityQuestion store
      fetchedSecurityQuestion.set(true); // Set fetchedSecurityQuestion to true
      // otpSent.set(true); // Set otpSent to true
    } catch (error) {
      console.error("There was a problem with the fetch operation:", error);
      securityQuestion.set("Failed to fetch security question");
    }
  }

  async function sendOTP() {
    try {
      const res = await fetch("http://127.0.0.1:5000/send-otp", {
        method: "POST",
        body: getFormData(),
      });

      if (!res.ok) {
        throw new Error("Network response was not ok");
      }
      otpSent.set(true); // Set otpSent to true
      // fetchedSecurityQuestion.set(true); // Set fetchedSecurityQuestion to true
      console.log($securityQuestion);
    } catch (error) {
      console.error("There was a problem with the fetch operation:", error);
      securityQuestion.set("Failed to fetch security question");
    }
  }

  async function downloadICSFile() {
    try {
      const res = await fetch("http://127.0.0.1:5000/verify-otp-and-download", {
        method: "POST",
        body: getFormData(),
      });

      if (!res.ok) {
        throw new Error("Network response was not ok");
      }

      const blob = await res.blob();
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "timetable.ics";
      document.body.appendChild(a);
      a.click();
      a.remove();
      URL.revokeObjectURL(url);

      formData.roll = "";
      formData.pass = "";
      formData.secret_answer = "";
      formData.email_otp = "";
      otpSent.set(false);
      securityQuestion.set("");
      fetchedSecurityQuestion.set(false);
    } catch (error) {
      console.error("There was a problem with the fetch operation:", error);
    }
  }
</script>

{#if !$fetchedSecurityQuestion}
  <div in:fade={{ duration: 300, delay: 0, easing: quintIn }}>
    <Header logoVisible={true} />
    <div class="input-container">
      <div>
        <h3>ERP Roll Number :</h3>
        <input
          bind:value={formData.roll}
          placeholder="Enter Your ERP Roll Number"
        />
      </div>
      <button on:click={getSecurityQuestion}>Get Security Question</button>
    </div>
    <Footer />
  </div>
{:else}
  <div in:fade={{ duration: 300, delay: 0, easing: quintIn }}>
    <Header logoVisible={false} />
    <div class="input-container">
      <div>
        <h3>ERP Roll Number :</h3>
        <input
          bind:value={formData.roll}
          placeholder="Enter Your ERP Roll Number"
        />
      </div>
      <div>
        <h3>{$securityQuestion} :</h3>
        <input
          bind:value={formData.secret_answer}
          type="password"
          placeholder="Security Answer"
        />
      </div>
      <div>
        <h3>Password :</h3>
        <input
          bind:value={formData.pass}
          type="password"
          placeholder="Password"
        />
      </div>
      {#if !$otpSent}
        <button on:click={sendOTP}>Send OTP</button>
      {:else}
        <div>
          <div>
            <h3>OTP :</h3>
            <input
              bind:value={formData.email_otp}
              type="password"
              placeholder="OTP"
            />
          </div>
          <button on:click={downloadICSFile}>Download ICS File</button>
        </div>
      {/if}
    </div>
    <Footer />
  </div>
{/if}

<style>
  .input-container {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
    border-bottom: 2px solid white;
    box-shadow: 0 20px 10px -25px #000d;
    margin-bottom: 1rem;
    padding-bottom: 1rem;
  }
  input {
    border: none;
    background-color: white;
    padding: 1rem;
    font-size: 1rem;
    border-radius: 8px;
    width: 100%;
    max-width: 20rem;
  }
  button {
    border: none;
    background-color: #00ba29;
    padding: 1rem;
    font-size: 1rem;
    border-radius: 8px;
    width: 100%;
    max-width: 20rem;
    color: white;
    font-weight: bold;
    cursor: pointer;
    margin-inline: auto;
  }
</style>
