<template>
  <div class="wrapper">
    <el-card class="card" shadow="always">
      <h2 class="title">🖼️ Image to Text (Khmer + English)</h2>

      <!-- Drag and Drop Zone -->
      <div class="drop-zone" @dragover.prevent @drop.prevent="handleDrop">
        <el-form-item label="Language">
          <el-select
            v-model="selectedLang"
            placeholder="Select language"
            style="width: 100%"
          >
            <el-option label="Khmer + English" value="eng+khm" />
            <el-option label="Khmer Only" value="khm" />
            <el-option label="English Only" value="eng" />
          </el-select>
        </el-form-item>

        <el-upload
          class="upload"
          drag
          :show-file-list="false"
          :auto-upload="false"
          accept="image/*"
          @change="handleFileChange"
          v-if="loading == false"
        >
          <el-icon><UploadFilled /></el-icon>
          <div class="el-upload__text">
            Drop file here or <em>click to upload</em>
          </div>
        </el-upload>
      </div>

      <!-- Show preview -->
      <div v-if="previewUrl" class="preview-container">
        <img :src="previewUrl" alt="Preview" class="preview-image" />
      </div>

      <!-- Loading Spinner -->
      <el-skeleton v-if="loading" animated />
      <div v-else class="result">
        <el-text
          type="textarea"
          placeholder="Recognized text will appear here..."
          rows="6"
          class="paragraph"
          >{{ text }}</el-text
        >
        <el-button
          type="success"
          class="copy-btn"
          :disabled="!text"
          @click="copyText"
        >
          Copy Text
        </el-button>
      </div>
    </el-card>
    <footer class="footer">
      Made with ❤️ by
      <a href="https://github.com/David-Deve" target="_blank">David</a>
    </footer>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { ElMessage } from "element-plus";
import { UploadFilled } from "@element-plus/icons-vue";
import { recognize } from "tesseract.js";
const selectedLang = ref("eng+khm");

const text = ref("");
const loading = ref(false);
const previewUrl = ref<string | null>(null);
const handleFileChange = async (file: any) => {
  const rawFile = file.raw || file;
  if (rawFile && rawFile.type.startsWith("image/")) {
    createPreview(rawFile);
    await recognizeTextFromImage(rawFile);
  }
};

const handleDrop = (e: DragEvent) => {
  const files = e.dataTransfer?.files;
  if (files && files.length > 0) {
    const file = files[0];
    if (file.type.startsWith("image/")) {
      createPreview(file);
      recognizeTextFromImage(file);
    } else {
      ElMessage.warning("Please drop an image file.");
    }
  }
};

const createPreview = (file: File) => {
  previewUrl.value = URL.createObjectURL(file);
};

const recognizeTextFromImage = async (imageFile: File) => {
  loading.value = true;
  text.value = "";

  try {
    const {
      data: { text: recognizedText },
    } = await recognize(imageFile, selectedLang.value, {
      logger: (m) => console.log(m),
    });

    text.value = recognizedText;
  } catch (err) {
    ElMessage.error("Failed to recognize text.");
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const copyText = async () => {
  try {
    await navigator.clipboard.writeText(text.value);
    ElMessage.success("Text copied!");
  } catch (err) {
    ElMessage.error("Failed to copy.");
  }
};
</script>

<style scoped>
.wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background: #f5f5f5;
}

.card {
  width: 500px;
  padding: 20px;
  text-align: center;
}

.drop-zone {
  margin-bottom: 20px;
}

.preview-container {
  margin-bottom: 20px;
  text-align: center;
}

.preview-image {
  max-width: 100%;
  max-height: 250px;
  border-radius: 8px;
  box-shadow: 0 0 5px #ccc;
}

.copy-btn {
  margin-top: 25px;
  float: none;
}
.footer {
  position: fixed;
  bottom: 10px;
  left: 0;
  right: 0;
  text-align: center;
  font-size: 14px;
  color: #888;
  margin-bottom: 20px;
}
.footer a {
  color: #409eff;
  text-decoration: none;
}
.footer a:hover {
  text-decoration: underline;
}
</style>
