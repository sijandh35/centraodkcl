<!--
Copyright 2017 ODK Central Developers
See the NOTICE file at the top-level directory of this distribution and at
https://github.com/getodk/central-frontend/blob/master/NOTICE.

This file is part of ODK Central. It is subject to the license terms in
the LICENSE file found in the top-level directory of this distribution and at
https://www.apache.org/licenses/LICENSE-2.0. No part of ODK Central,
including this file, may be copied, modified, propagated, or distributed
except according to the terms contained in the LICENSE file.
-->

<!-- FormAttachmentPopups used to contain multiple popups, but it now contains
only one. However, we have not changed its name to reflect that. It is also
possible that FormAttachmentPopups will need to contain a second popup again at
some point. -->
<template>
  <div id="form-attachment-popups" ref="popups" class="modal-dialog">
    <div v-show="state" id="form-attachment-popups-main" class="modal-content">
      <div class="modal-header">
        <span class="icon-cloud-upload"></span>
        <h4 class="modal-title">{{ $t('title') }}</h4>
      </div>
      <div class="modal-body">
        <template v-if="shownDuringDragover">
          <i18n-t v-if="dragoverAttachment != null" tag="p"
            keypath="duringDragover.dropToUpload">
            <template #attachmentName>
              <strong>{{ dragoverAttachment.name }}</strong>
            </template>
          </i18n-t>
          <p v-else-if="countOfFilesOverDropZone === 1">
            {{ $t('duringDragover.dragover') }}
          </p>
          <i18n-t v-else tag="p" keypath="duringDragover.dropToPrepare.full">
            <template #countOfFiles>
              <strong>{{ $tcn('duringDragover.dropToPrepare.countOfFiles', countOfFilesOverDropZone) }}</strong>
            </template>
          </i18n-t>
        </template>
        <template v-else-if="shownAfterSelection">
          <template v-if="plannedUploads.length !== 0">
            <i18n-t tag="p" keypath="afterSelection.matched.full"
              :plural="plannedUploads.length">
              <template #countOfFiles>
                <strong>{{ $tcn('afterSelection.matched.countOfFiles', plannedUploads.length) }}</strong>
              </template>
            </i18n-t>
            <p v-show="unmatchedFiles.length !== 0"
              id="form-attachment-popups-unmatched">
              <span class="icon-exclamation-triangle"></span>
              <i18n-t tag="span" keypath="afterSelection.someUnmatched.full"
                :plural="unmatchedFiles.length">
                <template #countOfFiles>
                  <strong>{{ $tcn('afterSelection.someUnmatched.countOfFiles', unmatchedFiles.length) }}</strong>
                </template>
              </i18n-t>
            </p>
            <p>
              <button type="button" class="btn btn-primary"
                @click="$emit('confirm')">
                {{ $t('action.looksGood') }}
              </button>
              <button type="button" class="btn btn-link"
                @click="$emit('cancel')">
                {{ $t('action.cancel') }}
              </button>
            </p>
          </template>
          <template v-else>
            <p>
              {{ $tc('afterSelection.noneMatched', unmatchedFiles.length) }}
            </p>
            <p>
              <button type="button" class="btn btn-primary"
                @click="$emit('cancel')">
                {{ $t('action.ok') }}
              </button>
            </p>
          </template>
        </template>
        <template v-else-if="shownDuringUpload">
          <p>{{ $tcn('duringUpload.total', uploadStatus.total) }}</p>
          <p id="form-attachment-popups-current">
            <strong>{{ $t('duringUpload.current', { filename: uploadStatus.current, percentUploaded }) }}</strong>
          </p>
          <p v-if="uploadStatus.total !== 1">
            <template v-if="uploadStatus.remaining > 1">
              {{ $tcn('duringUpload.remaining.beforeLast', uploadStatus.remaining) }}
            </template>
            <template v-else>
              {{ $t('duringUpload.remaining.last') }}
            </template>
          </p>
        </template>
      </div>
    </div>
    <div v-show="shownDuringUpload" id="form-attachment-popups-backdrop">
    </div>
  </div>
</template>

<script>
export default {
  name: 'FormAttachmentPopups',
  props: {
    countOfFilesOverDropZone: {
      type: Number,
      required: true
    },
    dragoverAttachment: Object,
    plannedUploads: {
      type: Array,
      required: true
    },
    unmatchedFiles: {
      type: Array,
      required: true
    },
    nameMismatch: {
      type: Object,
      required: true
    },
    uploadStatus: {
      type: Object,
      required: true
    }
  },
  emits: ['confirm', 'cancel'],
  computed: {
    shownDuringDragover() {
      return this.countOfFilesOverDropZone !== 0;
    },
    shownAfterSelection() {
      // If the user dropped a single file over a row, then
      // FormAttachmentPopups is not used to confirm the selection:
      // FormAttachmentNameMismatch is.
      return (this.plannedUploads.length !== 0 && !this.nameMismatch.state) ||
        this.unmatchedFiles.length !== 0;
    },
    shownDuringUpload() {
      return this.uploadStatus.current != null;
    },
    state() {
      return this.shownDuringDragover || this.shownAfterSelection ||
        this.shownDuringUpload;
    },
    percentUploaded() {
      const { progress } = this.uploadStatus;
      const fraction = progress != null && progress.lengthComputable
        ? progress.loaded / progress.total
        : 0;
      return this.$n(fraction, 'percent');
    }
  },
  updated() {
    if (this.shownAfterSelection)
      $(this.$refs.popups).find('.btn-primary').focus();
  }
};
</script>

<style lang="scss">
@import '../../assets/scss/variables';

$z-index-backdrop: 1;
$z-index-main: $z-index-backdrop + 1;

$edge-offset: 25px;
$popup-width: 300px;

#form-attachment-popups {
  position: absolute;
}

#form-attachment-popups-main {
  bottom: $edge-offset;
  position: fixed;
  right: $edge-offset;
  width: $popup-width;
  z-index: $z-index-main;

  .modal-header {
    background-color: $color-action-background;

    .icon-cloud-upload {
      animation-direction: alternate;
      animation-duration: 4s;
      animation-iteration-count: infinite;
      animation-name: bob;
      animation-timing-function: ease-in-out;
      color: #fff;
      font-size: 90px;
      position: absolute;
      right: 10px;
      top: -25px;

      $lin-offset: 4px;
      $diag-offset: 3px;
      text-shadow:
        #{-$lin-offset} 0 0 $color-action-overlay,
        0 #{-$lin-offset} 0 $color-action-overlay,
        #{$lin-offset}  0 0 $color-action-overlay,
        0 #{$lin-offset}  0 $color-action-overlay,
        #{-$diag-offset} #{-$diag-offset} 0 $color-action-overlay,
        #{$diag-offset}  #{-$diag-offset} 0 $color-action-overlay,
        #{-$diag-offset} #{$diag-offset}  0 $color-action-overlay,
        #{$diag-offset}  #{$diag-offset}  0 $color-action-overlay;

      &::after {
        background-color: $color-action-overlay;
        content: '';
        height: 45px;
        left: 20px;
        position: absolute;
        top: 20px;
        width: 45px;
        z-index: -1;
      }
    }
  }

  .modal-body {
    padding-bottom: 10px;

    #form-attachment-popups-unmatched {
      $padding: 10px;

      background-color: $color-warning-light;
      font-size: 12px;
      line-height: 14px;
      margin-bottom: 17px;
      padding: $padding;
      padding-left: 30px;
      position: relative;

      .icon-exclamation-triangle {
        left: $padding;
        margin-top: 1px;
        position: absolute;
      }
    }
  }
}

#form-attachment-popups-backdrop {
  background-color: #000;
  bottom: 0;
  left: 0;
  opacity: .8;
  position: fixed;
  right: 0;
  top: 0;
  z-index: $z-index-backdrop;
}

@keyframes bob {
  0% {
    transform: translateY(0);
  }
  100% {
    transform: translateY(-9px);
  }
}
</style>

<i18n lang="json5">
{
  "en": {
    // This is the title at the top of a pop-up.
    "title": "Upload Files",
    "duringDragover": {
      // {attachmentName} is formatted in bold.
      "dropToUpload": "Drop now to upload this file as {attachmentName}.",
      "dragover": "Drag over the Form Attachment you wish to replace with the file and drop to upload.",
      "dropToPrepare": {
        "full": "Drop now to prepare {countOfFiles} for upload to this Form.",
        "countOfFiles": "{count} file | {count} files"
      }
    },
    "afterSelection": {
      "matched": {
        "full": "{countOfFiles} ready for upload. | {countOfFiles} ready for upload.",
        "countOfFiles": "{count} file | {count} files"
      },
      "someUnmatched": {
        "full": "{countOfFiles} has a name we don???t recognize and will be ignored. To upload it, rename it or drag it onto its target. | {countOfFiles} have a name we don???t recognize and will be ignored. To upload them, rename them or drag them individually onto their targets.",
        "countOfFiles": "{count} file | {count} files"
      },
      "noneMatched": "We don???t recognize the file you are trying to upload. Please rename it to match the names listed above, or drag it individually onto its target. | We don???t recognize any of the files you are trying to upload. Please rename them to match the names listed above, or drag them individually onto their targets."
    },
    "duringUpload": {
      "total": "Please wait, uploading your {count} file: | Please wait, uploading your {count} files:",
      // Displayed in a pop-up to indicate a file that is currently being uploaded to be attached to a Form.
      "current": "Sending {filename} ({percentUploaded})",
      "remaining": {
        "beforeLast": "{count} file remains. | {count} files remain.",
        "last": "This is the last file."
      }
    }
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "title": "Nahr??t soubory",
    "duringDragover": {
      "dropToUpload": "P??eta??en??m nyn?? nahrajte tento soubor jako {attachmentName}.",
      "dragover": "P??et??hn??te p????lohu formul????e, kterou chcete nahradit souborem, a upu??t??n??m ji nahrajte.",
      "dropToPrepare": {
        "full": "P??et??hn??te nyn?? {countOfFiles} a p??ipravte se na upload do tohoto formul????e.",
        "countOfFiles": "{count} soubor | {count} soubor?? | {count} soubor?? | {count} soubor??"
      }
    },
    "afterSelection": {
      "matched": {
        "full": "{countOfFiles} p??ipraven k nahr??n?? | {countOfFiles} p??ipraveny k nahr??n?? | {countOfFiles} p??ipraveno k nahr??n?? | {countOfFiles} p??ipraveno k nahr??n??",
        "countOfFiles": "{count} soubor | {count} soubory | {count} soubor?? | {count} soubor??"
      },
      "someUnmatched": {
        "full": "{countOfFiles} m?? n??zev, kter?? nezn??me a bude ignorov??no. Chcete-li jej nahr??t, p??ejmenujte jej, nebo p??et??hn??te na sv??j c??l. | {countOfFiles} maj?? n??zev, kter?? nezn??me a budou ignorov??ny. Chcete-li je nahr??t, p??ejmenujte je, nebo p??et??hn??te na sv??j c??l. | {countOfFiles} maj?? n??zev, kter?? nezn??me a budou ignorov??ny. Chcete-li je nahr??t, p??ejmenujte je, nebo p??et??hn??te na sv??j c??l. | {countOfFiles} maj?? n??zev, kter?? nezn??me a budou ignorov??ny. Chcete-li je nahr??t, p??ejmenujte je, nebo p??et??hn??te na sv??j c??l.",
        "countOfFiles": "{count} soubor | {count} soubory | {count} soubory | {count} soubor??"
      },
      "noneMatched": "Nerozpozn??v??me soubor, kter?? se pokou????te nahr??t. P??ejmenujte ho tak, aby odpov??dal v????e uveden??m n??zv??m, nebo ho p??et??hn??te jednotliv?? na jeho c??l. | Nerozpozn??v??me soubor, kter?? se pokou????te nahr??t. P??ejmenujte ho tak, aby odpov??dal v????e uveden??m n??zv??m, nebo ho p??et??hn??te jednotliv?? na jeho c??l. | Nerozpozn??v??me soubor, kter?? se pokou????te nahr??t. P??ejmenujte ho tak, aby odpov??dal v????e uveden??m n??zv??m, nebo ho p??et??hn??te jednotliv?? na jeho c??l. | Nerozpozn??v??me ????dn?? ze soubor??, kter?? se pokou????te nahr??t. P??ejmenujte je podle v????e uveden??ch n??zv?? nebo je p??et??hn??te jednotliv?? na jejich c??le."
    },
    "duringUpload": {
      "total": "??ekejte pros??m, nahr??v??me {count} soubor: | ??ekejte pros??m, nahr??v??me {count} soubory: | ??ekejte pros??m, nahr??v??me {count} soubor??: | ??ekejte pros??m, nahr??v??me {count} soubor??:",
      "current": "Odes??l??n?? {filename} ({percentUploaded})",
      "remaining": {
        "beforeLast": "{count} soubor zb??v??. | {count} soubory zb??vaj??. | {count} soubor?? zb??v??. | {count} soubor?? zb??v??.",
        "last": "Toto je posledn?? soubor."
      }
    }
  },
  "de": {
    "title": "Dateien hochladen",
    "duringDragover": {
      "dropToUpload": "Jetzt loslassen, um die Datei als {attachmentName} hochzuladen.",
      "dropToPrepare": {
        "full": "Lassen Sie die {countOfFiles} hier fallen, um sie hochzuladen.",
        "countOfFiles": "{count} Datei | {count} Dateien"
      }
    },
    "afterSelection": {
      "matched": {
        "full": "{countOfFiles} bereit zum Hochladen | {countOfFiles} bereit zum Hochladen",
        "countOfFiles": "{count} Datei | {count} Dateien"
      },
      "someUnmatched": {
        "full": "{countOfFiles} Datei hat einen Namen, den wir nicht erkennen, sie wird ignoriert. Um sie hochzuladen, bitte umbenennen oder einzeln auf ihr Ziel ziehen. | {countOfFiles} Dateien haben Namen, die wir nicht erkennen, diese werden ignoriert. Um sie hochzuladen, bitte umbenennen oder einzeln auf ihr Ziel ziehen.",
        "countOfFiles": "{count} Datei | {count} Dateien"
      },
      "noneMatched": "Wir erkennen keine der Dateien, die Sie hochzuladen versuchen. Bitte benennen Sie Dateien um, damit sie mit den oben gelisteten Namen ??bereinstimmen, oder ziehen Sie sie einzeln auf ihre Ziele. | Wir erkennen keine der Dateien, die Sie hochzuladen versuchen. Bitte benennen Sie Dateien um, damit sie mit den oben gelisteten Namen ??bereinstimmen, oder ziehen Sie sie einzeln auf ihre Ziele."
    },
    "duringUpload": {
      "total": "Bitte warten Sie, {count} Datei wird hochgeladen. | Bitte warten Sie, {count} Dateien werden hochgeladen.",
      "current": "Sende {filename} ({percentUploaded})",
      "remaining": {
        "beforeLast": "Noch {count} Datei | Noch {count} Dateien",
        "last": "Dies ist die letzte Datei"
      }
    }
  },
  "es": {
    "title": "Subir archivos",
    "duringDragover": {
      "dropToUpload": "Suelta ahora para cargar este archivo como {attachmentName}",
      "dropToPrepare": {
        "full": "Suelte ahora para preparar {countOfFiles} para subir a este formulario.",
        "countOfFiles": "{count} archivo | {count} archivos | {count} archivos"
      }
    },
    "afterSelection": {
      "matched": {
        "full": "{countOfFiles} listo para subir. | {countOfFiles} listos para subir. | {countOfFiles} listos para subir.",
        "countOfFiles": "{count} | {count} archivos | {count} archivos"
      },
      "someUnmatched": {
        "full": "{countOfFiles} tiene un nombre que no reconocemos y ser?? ignorado. Para cargarlo, c??mbiele el nombre o arr??strelo a su destino. | {countOfFiles} tiene un nombre que no reconocemos y ser?? ignorado. Para cargarlo, c??mbiele el nombre o arr??strelo a su destino. | {countOfFiles} tiene un nombre que no reconocemos y ser?? ignorado. Para cargarlo, c??mbiele el nombre o arr??strelo a su destino.",
        "countOfFiles": "{count} archivos | {count} archivos | {count} archivos"
      },
      "noneMatched": "No reconocemos ninguno de los archivos que est?? intentando cargar. Cambie el nombre de ellos para que coincidan con los nombres enumerados anteriormente, o arr??strelos individualmente a sus objetivos. | No reconocemos ninguno de los archivos que est?? intentando cargar. Cambie el nombre de ellos para que coincidan con los nombres enumerados anteriormente, o arr??strelos individualmente a sus objetivos. | No reconocemos ninguno de los archivos que est?? intentando cargar. Cambie el nombre de ellos para que coincidan con los nombres enumerados anteriormente, o arr??strelos individualmente a sus objetivos."
    },
    "duringUpload": {
      "total": "Por favor espere, cargando su {count} archivo: | Por favor espere, cargando sus {count} archivos: | Por favor espere, cargando sus {count} archivos:",
      "current": "Enviando {filename} ({percentUploaded})",
      "remaining": {
        "beforeLast": "{count} archivo permanece. | {count} archivos permanecen. | {count} archivos permanecen.",
        "last": "Este es el ??ltimo archivo."
      }
    }
  },
  "fr": {
    "title": "T??l??verser des fichiers",
    "duringDragover": {
      "dropToUpload": "D??poser maintenant pour t??l??verser ce fichier en tant que {attachmentName}.",
      "dropToPrepare": {
        "full": "D??posez maintenant pour pr??parer le t??l??versement de {countOfFiles} vers ce formualire.",
        "countOfFiles": "{count} fichier | {count} fichiers | {count} fichiers"
      }
    },
    "afterSelection": {
      "matched": {
        "full": "{countOfFiles} pr??ts ?? ??tre t??l??vers??s for upload. | {countOfFiles} pr??ts ?? ??tre t??l??vers??s. | {countOfFiles} pr??ts ?? ??tre t??l??vers??s.",
        "countOfFiles": "{count} fichier | {count} fichiers | {count} fichiers"
      },
      "someUnmatched": {
        "full": "{countOfFiles} a un nom que nous ne reconnaissons pas et qui sera ignor??. Pour le t??l??verser, renommez le ou d??posez le sur sa cible. | {countOfFiles} ont un nom que nous ne reconnaissons pas et qui sera ignor??. Pour les t??l??verser, renommez les ou d??posez les un ?? un sur leurs cibles. | {countOfFiles} ont un nom que nous ne reconnaissons pas et qui sera ignor??. Pour les t??l??verser, renommez les ou d??posez les un ?? un sur leurs cibles.",
        "countOfFiles": "{count} fichier | {count} fichiers | {count} fichiers"
      },
      "noneMatched": "Nous ne reconnaissons pas le fichier que vous tentez de t??l??verser. Renommez le pour qu'il corresponde aux noms listez ci-dessous, ou d??posez le individuellement sur sa cible. | Nous ne reconnaissons aucun des fichiers que vous tentez de t??l??verser. Renommez les pour qu'ils correspondent aux noms listez ci-dessous, ou d??posez les un ?? un sur leurs cibles. | Nous ne reconnaissons aucun des fichiers que vous tentez de t??l??verser. Renommez les pour qu'ils correspondent aux noms listez ci-dessous, ou d??posez les un ?? un sur leurs cibles."
    },
    "duringUpload": {
      "total": "Merci de patienter, t??l??versement de votre {count} fichier en cours : | Merci de patienter, t??l??versement de vos {count} fichiers en cours : | Merci de patienter, t??l??versement de vos {count} fichiers en cours :",
      "current": "Envoi de {filename} ({percentUploaded})",
      "remaining": {
        "beforeLast": "{count} fichier restant. | {count} fichiers restants. | {count} fichiers restants.",
        "last": "Ceci est le dernier fichier."
      }
    }
  },
  "id": {
    "title": "Unggah Berkas",
    "duringDragover": {
      "dropToUpload": "Lepas sekarang untuk mengunggah dokumen ini dengan nama {attachmentName}.",
      "dropToPrepare": {
        "full": "Lepas untuk mempersiapkan unggahan {countOfFiles} ke formulir ini.",
        "countOfFiles": "{count} berkas"
      }
    },
    "afterSelection": {
      "matched": {
        "full": "{countOfFiles} siap untuk diunggah.",
        "countOfFiles": "{count} berkas"
      },
      "someUnmatched": {
        "full": "Nama {countOfFiles} tidak dikenali dan ditolak. Untuk mengunggah dokumen, ubah nama dokumen atau seret dokumen satu per satu ke target masing-masing.",
        "countOfFiles": "{count} berkas"
      },
      "noneMatched": "Kami tidak mengenali dokumen yang ingin Anda unggah. Mohon ubah nama dokumen sesuai dengan nama-nama yang terdaftar di atas, atau seret dokumen satu per satu ke targetnya masing-masing."
    },
    "duringUpload": {
      "total": "Mohon tungguh, menunggah {count} dokumen:",
      "current": "Mengirim {filename} ({percentUploaded})",
      "remaining": {
        "beforeLast": "Tersisa {count} dokumen.",
        "last": "Ini adalah dokumen terakhir."
      }
    }
  },
  "it": {
    "title": "Caricare files",
    "duringDragover": {
      "dropToUpload": "Rilascia ora per caricare questo file come {attachmentName}.",
      "dropToPrepare": {
        "full": "Rilascia ora per preparare {countOfFiles} per il caricamento in questo formulario.",
        "countOfFiles": "{count} file | {count} files | {count} files"
      }
    },
    "afterSelection": {
      "matched": {
        "full": "{countOfFiles} prontio per essere caricato. | {countOfFiles} pronti per essere caricati. | {countOfFiles} pronti per essere caricati.",
        "countOfFiles": "{count} file | {count} files | {count} files"
      },
      "someUnmatched": {
        "full": "{countOfFiles} ha un nome che non riconosciamo e che verra ignorato. Per caricarlo, rinominalo o trascinalo individualmente sul suo obiettivo. | {countOfFiles} hanno un nome che non riconosciamo e che verranno ignorati. Per caricarli, rinominali o trascinali individualmente sui loro obiettivi. | {countOfFiles} hanno un nome che non riconosciamo e che verranno ignorati. Per caricarli, rinominali o trascinali individualmente sui loro obiettivi.",
        "countOfFiles": "{count} file | {count} files | {count} files"
      },
      "noneMatched": "Non riconosciamo il file che stai tentando di caricare. Rinominalo in modo che corrisponda al nome elencao sopra o trascinalp individualmente sul suo obiettivo. | Non riconosciamo nessuno dei file che stai tentando di caricare. Rinominali in modo che corrispondano ai nomi elencati sopra o trascinali individualmente sui loro obiettivi. | Non riconosciamo nessuno dei file che stai tentando di caricare. Rinominali in modo che corrispondano ai nomi elencati sopra o trascinali individualmente sui loro obiettivi."
    },
    "duringUpload": {
      "total": "Attendere prego, caricando il tuo {count} file: | Attendere prego, caricando i tuoi {count} files: | Attendere prego, caricando i tuoi {count} files:",
      "current": "Inviando {filename} ({percentUploaded})",
      "remaining": {
        "beforeLast": "{count} file rimasto. | {count} files rimanenti. | {count} files rimanenti.",
        "last": "Questo ?? l'ultimo file"
      }
    }
  },
  "ja": {
    "title": "?????????????????????????????????",
    "duringDragover": {
      "dropToUpload": "?????????????????????{attachmentName}????????????????????????????????????",
      "dropToPrepare": {
        "full": "?????????????????????????????????????????????{countOfFiles}?????????????????????????????????????????????????????????",
        "countOfFiles": "{count}??????????????????"
      }
    },
    "afterSelection": {
      "matched": {
        "full": "{countOfFiles}????????????????????????????????????",
        "countOfFiles": "{count}??????????????????"
      },
      "someUnmatched": {
        "full": "{countOfFiles}???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????",
        "countOfFiles": "{count}??????????????????"
      },
      "noneMatched": "?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????"
    },
    "duringUpload": {
      "total": "????????????????????????{count}???????????????????????????????????????",
      "current": "{filename}???????????????{percentUploaded}???",
      "remaining": {
        "beforeLast": "{count}??????????????????????????????????????????",
        "last": "??????????????????"
      }
    }
  },
  "sw": {
    "title": "Pakia Faili",
    "duringDragover": {
      "dropToUpload": "Dondosha sasa ili upakie faili hii kama {attachmentName}.",
      "dropToPrepare": {
        "full": "Dondosha sasa ili kuandaa {countOfFiles} kwa ajili ya kupakiwa kwenye Fomu hii.",
        "countOfFiles": "faili {count} | faili {count}"
      }
    },
    "afterSelection": {
      "matched": {
        "full": "{countOfFiles} tayari kupakiwa. | {countOfFiles} tayari kupakiwa.",
        "countOfFiles": "faili {count} | faili {count}"
      },
      "someUnmatched": {
        "full": "{countOfFiles} wana jina ambalo hatulitambui na tutapuuzwa. Ili kuzipakia, zipe jina jipya au ziburute kibinafsi hadi kwenye malengo yao | {countOfFiles} wana jina ambalo hatulitambui na tutapuuzwa. Ili kuzipakia, zipe jina jipya au ziburute kibinafsi hadi kwenye malengo yao.",
        "countOfFiles": "faili {count} | faili {count}"
      },
      "noneMatched": "Hatutambui faili zozote unazojaribu kupakia. Tafadhali zipe jina jipya ili zilingane na majina yaliyoorodheshwa hapo juu, au ziburute moja moja hadi kwenye malengo yao | Hatutambui faili zozote unazojaribu kupakia. Tafadhali zipe jina jipya ili zilingane na majina yaliyoorodheshwa hapo juu, au ziburute moja moja hadi kwenye malengo yao"
    },
    "duringUpload": {
      "total": "Tafadhali subiri, faili yako inapakiwa {count}: | Tafadhali subiri, faili zako zinapakiwa {count}:",
      "current": "Inatuma {filename} ({percentUploaded})",
      "remaining": {
        "beforeLast": "faili {count} itasalia. | faili {count} zitasalia.",
        "last": "Hili ndilo faili la mwisho."
      }
    }
  }
}
</i18n>
