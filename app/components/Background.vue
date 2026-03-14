<template>
  <div class="jp-matrix">
    <span
      v-for="(char, i) in characters"
      :key="i"
      :style="{
        animationDelay: `${Math.random() * 5}s`,
        animationDuration: `${2 + Math.random() * 4}s`,
        opacity: Math.random(),
      }"
      >{{ char }}</span
    >
  </div>
</template>

<script setup>
const kana = [
  "ア",
  "イ",
  "ウ",
  "エ",
  "オ",
  "カ",
  "キ",
  "ク",
  "ケ",
  "コ",
  "サ",
  "シ",
  "ス",
  "セ",
  "ソ",
  "タ",
  "チ",
  "ツ",
  "テ",
  "ト",
  "ナ",
  "ニ",
  "ヌ",
  "ネ",
  "ノ",
  "ハ",
  "ヒ",
  "フ",
  "ヘ",
  "ホ",
  "マ",
  "ミ",
  "ム",
  "メ",
  "モ",
  "ヤ",
  "ユ",
  "ヨ",
  "ラ",
  "リ",
  "ル",
  "レ",
  "ロ",
  "ワ",
  "ヲ",
  "ン",
];

const characters = Array.from(
  { length: 800 },
  () => kana[Math.floor(Math.random() * kana.length)],
);
</script>
