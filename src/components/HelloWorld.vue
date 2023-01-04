<script setup lang="ts">
import { Octokit, App } from "https://cdn.skypack.dev/octokit";
import { decode,encode } from 'js-base64';
import { onMounted, ref,computed } from "vue";
import { marked } from 'marked'

defineProps<{ msg: string }>();

const count = ref(0);

const octokit = new Octokit({
  auth: "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",
});
const owner = "qixiaobro",
    repo = "test-git-note",
    perPage = 5;
const getCommit = async () => {


  const fiveMostRecentCommits = await octokit.request(
    `GET /repos/{owner}/{repo}/commits`,
    { owner, repo, per_page: perPage }
  );
  console.log(fiveMostRecentCommits);
};

// 获取文件树
const getTrees = async () => {
  const fiveMostRecentCommits = await octokit.request(
    `GET /repos/{owner}/{repo}/git/trees/{tree_sha}{?recursive}{?timestamp}`,
    { owner, repo, tree_sha: "main", recursive: 1,timestamp: new Date().getTime() }
  );
  console.log(fiveMostRecentCommits);
};

const content = ref("");
const getFile = async () => {
  const res = await octokit.request(
    `GET /repos/{owner}/{repo}/contents/{path}`,
    { owner, repo, path: "README.md" }
  );
  content.value = decode(res.data.content);
};

const output = computed(() => marked(content.value))

const createBlob = async () => {
  const res = await octokit.request(
    `POST /repos/{owner}/{repo}/git/blobs`,
    { owner, repo, content: encode("hello world"), encoding: "base64" }
  );
  console.log(res);
};

const fileName = ref('')

// 创建一个文件，并提交
const createFile = async () => {
  const res = await octokit.request(
    `PUT /repos/{owner}/{repo}/contents/{path}`,
    { owner, repo, path: fileName.value,message:'update', content: encode("hello world"), encoding: "base64" }
  );
  console.log(res);
  getTrees()
};

onMounted(() => {
  getTrees();
});
</script>

<template>
 <article v-html="output"></article>
 <input v-model="fileName"/>
 <button @click="createFile">create a file</button>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
