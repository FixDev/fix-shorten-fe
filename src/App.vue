<script setup>
import { ref } from 'vue';
import { useQuasar } from 'quasar';

const $q = useQuasar();

const originUrl = ref('');
const useCustomLength = ref(false);
const lengthUrl = ref(6);

const showGeneratedUrl = ref(false);
const generatedUrl = ref('');

const isURL = (value) => {
  return /^(?:(?:(?:https?|ftp):)?\/\/)(?:\S+(?::\S*)?@)?(?:(?!(?:10|127)(?:\.\d{1,3}){3})(?!(?:169\.254|192\.168)(?:\.\d{1,3}){2})(?!172\.(?:1[6-9]|2\d|3[0-1])(?:\.\d{1,3}){2})(?:[1-9]\d?|1\d\d|2[01]\d|22[0-3])(?:\.(?:1?\d{1,2}|2[0-4]\d|25[0-5])){2}(?:\.(?:[1-9]\d?|1\d\d|2[0-4]\d|25[0-4]))|(?:(?:[a-z0-9\u00a1-\uffff][a-z0-9\u00a1-\uffff_-]{0,62})?[a-z0-9\u00a1-\uffff]\.)+(?:[a-z\u00a1-\uffff]{2,}\.?))(?::\d{2,5})?(?:[/?#]\S*)?$/i.test(
    value
  );
};

const onGenerated = async () => {
  $q.loading.show({
    message: 'Sedang mengenerate url, harap tunggu...',
  });
  showGeneratedUrl.value = false;

  const req = {
    origUrl: originUrl.value,
    useCustomUrl: useCustomLength.value,
    howMuch: parseInt(lengthUrl.value),
  };

  try {
    const res = await fetch(
      'https://fix-shorten-production.up.railway.app/api/short',
      {
        method: 'POST',
        mode: 'cors',
        body: JSON.stringify(req),
        headers: {
          'Content-Type': 'application/json',
          // 'Content-Type': 'application/x-www-form-urlencoded',
        },
        cache: 'no-cache',
      }
    );
    const _res = await res.json();

    showGeneratedUrl.value = true;
    generatedUrl.value = _res.shortUrl;
    $q.loading.hide();
    $q.dialog({
      title: 'Hasil URL',
      message: 'Selamat, anda berhasil mengenerate URL Baru :)',
      prompt: {
        model: generatedUrl.value,
        readonly: true,
        type: 'text', // optional
        dense: true,
        filled: true,
      },
      persistent: true,
      fullWidth: true,
    }).onOk((data) => {
      navigator.clipboard.writeText(data);
      $q.notify({
        type: 'positive',
        message: 'Berhasil mencopy URL kedalam clipboard!',
        position: 'top',
        actions: [
          {
            icon: 'close',
            color: 'white',
            round: true,
            handler: () => {
              /* ... */
            },
          },
        ],
        timeout: 5000,
      });
    });
    return;
  } catch (err) {
    showGeneratedUrl.value = false;
    $q.loading.hide();
    $q.notify({
      type: 'negative',
      message: 'Gagal Mengenerate URL',
      position: 'top',
      actions: [
        {
          icon: 'close',
          color: 'white',
          round: true,
          handler: () => {
            /* ... */
          },
        },
      ],
      timeout: 5000,
    });
    console.error(err);
  }
};
</script>

<template>
  <q-layout view="lHh lpr lFf" container style="height: 100vh" class="shadow-2">
    <q-header elevated flat>
      <q-toolbar>
        <q-avatar>
          <img src="https://cdn.quasar.dev/logo-v2/svg/logo-mono-white.svg" />
        </q-avatar>

        <q-toolbar-title>Fix Shorten</q-toolbar-title>

        <q-btn flat round dense icon="info" />
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="q-pa-lg">
        <q-form @submit.prevent="onGenerated" class="column">
          <div class="col">
            <q-input
              filled
              bottom-slots
              v-model="originUrl"
              label="URL/Link Original"
              dense
              :rules="[
                (val) => !!val || 'Wajib diisi!!',
                (val) =>
                  isURL(val) ||
                  'Masukan URL yang valid seperti (https://google.com)',
              ]"
            >
              <template v-slot:append>
                <q-icon
                  v-if="originUrl !== ''"
                  name="close"
                  @click="originUrl = ''"
                  class="cursor-pointer"
                />
              </template>

              <template v-slot:hint> https://google.com </template>
            </q-input>
          </div>
          <div class="col">
            <q-toggle
              v-model="useCustomLength"
              color="primary"
              keep-color
              label="Gunakan panjang kustom url?"
              left-label
            />
          </div>
          <div class="col" v-if="useCustomLength">
            <q-input
              label="Panjang URL"
              v-model.number="lengthUrl"
              type="number"
              filled
              style="max-width: 120px"
              dense
              hint="Masukan 6-21"
              re
              :rules="[
                (val) =>
                  (val >= 6 && val <= 21) || 'Minimal 6 dan Maximal hanya 21',
              ]"
            />
          </div>
          <div class="col self-end q-pt-sm">
            <q-btn label="Generate URL" color="primary" type="submit" />
          </div>
        </q-form>
      </q-page>
    </q-page-container>
  </q-layout>
</template>

<style scoped></style>
