<script setup>
import { ref } from 'vue';
import { useQuasar } from 'quasar';

const $q = useQuasar();

const originUrl = ref('');
const useCustomLength = ref('');
const lengthUrl = ref(6);
const customUrl = ref('');

const showGeneratedUrl = ref(false);
const generatedUrl = ref('');

const isURL = (value) => {
  return /^(?:(?:(?:https?|ftp):)?\/\/)(?:\S+(?::\S*)?@)?(?:(?!(?:10|127)(?:\.\d{1,3}){3})(?!(?:169\.254|192\.168)(?:\.\d{1,3}){2})(?!172\.(?:1[6-9]|2\d|3[0-1])(?:\.\d{1,3}){2})(?:[1-9]\d?|1\d\d|2[01]\d|22[0-3])(?:\.(?:1?\d{1,2}|2[0-4]\d|25[0-5])){2}(?:\.(?:[1-9]\d?|1\d\d|2[0-4]\d|25[0-4]))|(?:(?:[a-z0-9\u00a1-\uffff][a-z0-9\u00a1-\uffff_-]{0,62})?[a-z0-9\u00a1-\uffff]\.)+(?:[a-z\u00a1-\uffff]{2,}\.?))(?::\d{2,5})?(?:[/?#]\S*)?$/i.test(
    value
  );
};

const hitOnlyShort = async () => {
  const req = {
    origUrl: originUrl.value,
    useCustomUrl: useCustomLength.value,
    howMuch: parseInt(lengthUrl.value),
  };
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
  return res.json();
};
const hitShortWithKustom = async () => {
  const req = {
    origUrl: originUrl.value,
    useCustomUrl: useCustomLength.value,
    customUrl: customUrl.value,
  };
  const res = await fetch(
    'https://fix-shorten-production.up.railway.app/api/short-custom',
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
  return res.json();
};

const onGenerated = async () => {
  $q.loading.show({
    message: 'Sedang mengenerate url, harap tunggu...',
  });
  showGeneratedUrl.value = false;

  try {
    const res =
      useCustomLength === 'Acak'
        ? await hitOnlyShort()
        : await hitShortWithKustom();

    showGeneratedUrl.value = true;
    generatedUrl.value = res.shortUrl;
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

        <q-btn flat round dense icon="info">
          <q-tooltip> This website can help you to short your URL! </q-tooltip>
        </q-btn>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="q-pa-xl">
        <q-form @submit.prevent="onGenerated" class="column">
          <div class="col">
            <div class="row q-col-gutter-xs">
              <div class="col-xs-12 col-sm-12 col-md-8">
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
              <div class="col-xs-12 col-sm-12 col-md-4">
                <q-select
                  dense
                  filled
                  :options="['Kustom', 'Acak']"
                  v-model="useCustomLength"
                  label="Jenis Kustom"
                />
              </div>
            </div>
          </div>
          <div class="col q-pt-md" v-if="useCustomLength === 'Acak'">
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
          <div class="col q-pt-md" v-if="useCustomLength === 'Kustom'">
            <q-input
              label="Kustom Url"
              v-model="customUrl"
              filled
              style="max-width: 500px"
              dense
              :rules="[
                (val) =>
                  (val.length >= 3 && val.length <= 21) ||
                  'Minimal 3 Karakter dan Maximal hanya 21 Karakter',
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
