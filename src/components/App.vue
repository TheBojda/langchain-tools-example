<template>
  <main role="main" class="container">
    <div style="padding-top: 1rem" class="d-none d-lg-block"></div>
    <div class="row" style="margin-bottom: 5px">
      <div class="col-sm-10">
        <div class="scrollable-div" v-html="log"></div>
      </div>
    </div>
    <div class="row justify-content-md-center">
      <div class="form-horizontal">
        <div class="form-group row">
          <div class="col-sm-8">
            <input
              type="text"
              class="form-control"
              id="input-box"
              placeholder="Enter text"
              v-model="message"
            />
          </div>
          <div class="col-sm-2">
            <button class="btn btn-primary" @click="submit">Submit</button>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script lang="ts">
import { Component, Vue } from "vue-facing-decorator";
import { OpenAI } from "langchain/llms/openai";
import { initializeAgentExecutor, AgentExecutor } from "langchain/agents";
import { Calculator } from "langchain/tools/calculator";

@Component
export default class App extends Vue {
  log: string = "";
  message: string = "";
  executor: AgentExecutor;

  mounted() {
    this.init();
  }

  async init() {
    const model = new OpenAI({
      temperature: 0,
      openAIApiKey: process.env.OPENAI_API_KEY,
    });
    const tools = [new Calculator()];

    this.executor = await initializeAgentExecutor(
      tools,
      model,
      "zero-shot-react-description"
    );

    this.writeLog("Agent loaded.");
  }

  writeLog(message: string) {
    this.log += `<p>${message}</p>`;
  }

  async submit() {
    this.writeLog(`User: ${this.message}`);
    const result = await this.executor.call({ input: this.message });

    this.writeLog(`AI: ${result.output}`);
    this.message = "";
  }
}
</script>

<style scoped>
.scrollable-div {
  border: 1px solid gray;
  height: 400px;
  width: 100%;
  overflow-y: scroll;
}
</style>