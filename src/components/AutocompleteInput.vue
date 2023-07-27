<template>
  <div class="formula-input" @keydown="handleKeyDown">
    <template v-for="(tag, index) in tags" :key="index">
      <span
        v-if="tag !== ''"
        class="tag"
        :class="{'option-chip': !isInOptions(tag)}"
        :ref="(el) => (tagRefs[index] = el)"
        @click="editTag(index)"
        @blur="saveTag(index)"
        contenteditable="true"
      >
        <!-- <span v-if="isInOptions(tag)"> {{ tag }}</span> -->
        <span>{{ tag }}</span>
        <span v-if="isInOptions(tag)" @click="removeTag(index)" class="cross-button">[x]</span>
      </span>
    </template>
    <input
      ref="inputRef"
      type="text"
      class="operand"
      v-model="currentOperand"
      @input="handleOperandInput"
    />
    <div v-if="showAutocomplete" class="autocomplete">
      <ul>
        <li
          v-for="(option, index) in autocompleteOptions"
          :key="index"
          @click="selectAutocompleteOption(option)"
          :class="{ 'active-list': activeOptionIndex === index }"
        >
          {{ option }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, nextTick } from "vue";

const tagRefs = ref<any[]>([]);
const tags = ref<string[]>([""]); // Initial tag array with an empty tag
const currentOperand = ref<string>("");
const showAutocomplete = ref<boolean>(false);
const autocompleteOptions = ["red", "blue", "green", "yellow", "purple"];
const inputRef = ref<HTMLInputElement | null>(null);

const activeOptionIndex = ref<number>(-1);

function handleKeyDown(event: KeyboardEvent) {
  const { key } = event;
  if (key === "Backspace") {
    handleBackspace();
  } else if (key === "Enter") {
    handleEnter();
  } else if (key === "+" || key === "-" || key === "*" || key === "/") {
    handleOperand(key);
  }
}

function handleBackspace() {
  const lastTag = tags.value[tags.value.length - 1];

  if (lastTag !== '') {
    if (isInOptions(lastTag)) {
      tags.value.pop();
    } else {
      const lastSpaceIndex = lastTag.lastIndexOf(' ');
      if (lastSpaceIndex !== -1) {
        // Remove the last word of the plain text
        tagRefs.value[tags.value.length - 1].innerText = lastTag.slice(0, lastSpaceIndex);
        // If the plain text becomes empty, remove the entire chip
        if (tagRefs.value[tags.value.length - 1].innerText === '') {
          tags.value.pop();
          tagRefs.value.pop();
        }
      } else {
        // If there are no more words, remove the entire chip
        tags.value.pop();
        tagRefs.value.pop();
      }
    }
  } else if (tags.value.length > 1) {
    tags.value.pop();
  }
}


function handleEnter() {
  const trimmedOperand = currentOperand.value.trim();

  if (trimmedOperand !== "") {
    // Check if the entered operand is in the autocompleteOptions list
    const foundOption = autocompleteOptions.find((option) => option === trimmedOperand);

    if (foundOption) {
      // If the entered operand is found in options, add it to the last tag
      const lastTag = tags.value[tags.value.length - 1];
      tags.value.splice(tags.value.length - 1, 1, lastTag + foundOption, "");
    } else {
      // If the entered operand is not in options, create a new tag for it
      tags.value.push(trimmedOperand);
    }
  } else {
    tags.value.push("");
  }
  currentOperand.value = "";
}

function handleOperand(operand: string) {
  const lastTag = tags.value[tags.value.length - 1];
  tags.value.splice(tags.value.length - 1, 1, lastTag + operand, "");
}

function handleOperandInput() {
  showAutocomplete.value = true;
}

function isInOptions(tag: string) {
  return autocompleteOptions.includes(tag);
}

function selectAutocompleteOption(option: string) {
  currentOperand.value = option;
  showAutocomplete.value = false;
  inputRef.value?.focus();
}

async function editTag(index: number) {
  const tagRef = tagRefs.value[index];
  if (tagRef) {
    tagRef.setAttribute("contenteditable", "true");
    tagRef.focus();

    // Wait for nextTick to ensure the tagRef is still valid before setting selection
    await nextTick();
    const selection = window.getSelection();
    const range = document.createRange();
    range.selectNodeContents(tagRef);
    selection?.removeAllRanges();
    selection?.addRange(range);
  }
}

function removeTag(index: number) {
  if (tagRefs.value[index]) {
    tags.value.splice(index, 1);
    tagRefs.value.splice(index, 1);
  }
}

function saveTag(index: number) {
  const tagRef = tagRefs.value[index];
  if (tagRef) {
    const tag = tagRef.innerText;
    tagRef.setAttribute("contenteditable", "false");
    tags.value.splice(index, 1, tag);
  }
}
</script>

<style>
.formula-input {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 4px;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-family: Arial, sans-serif;
}

.tag {
  display: inline-block;
  padding: 4px 8px;
  background-color: #f2f2f2;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
}
.option-chip {
  background-color: transparent;
  border:none;
}

.tag:focus {
  outline: none;
  border-color: #007bff;
}

.operand {
  flex: 1;
  padding: 10px 12px;
  background-color: transparent;
  border: none;
  outline: none;
}

.autocomplete {
  position: absolute;
  z-index: 1;
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1);
}

.autocomplete ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

.autocomplete li {
  padding: 4px 8px;
  cursor: pointer;
}
.cross-button {
  margin: 0 10px;
}

.active-list {
  background-color: #bbb7b7;
}

.autocomplete li:hover {
  background-color: #f2f2f2;
}
</style>
