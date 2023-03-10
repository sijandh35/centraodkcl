<!--
Copyright 2020 ODK Central Developers
See the NOTICE file at the top-level directory of this distribution and at
https://github.com/getodk/central-frontend/blob/master/NOTICE.

This file is part of ODK Central. It is subject to the license terms in
the LICENSE file found in the top-level directory of this distribution and at
https://www.apache.org/licenses/LICENSE-2.0. No part of ODK Central,
including this file, may be copied, modified, propagated, or distributed
except according to the terms contained in the LICENSE file.
-->
<template>
  <div v-if="dataExists">
    <checklist-step v-if="form.publishedAt == null" stage="complete">
      <template #title>{{ $t('steps[0].title') }}</template>
      <p>
        <strong>{{ $t('steps[0].body[0]') }}</strong>
        <sentence-separator/>
        <span>{{ $t('steps[0].body[1]') }}</span>
      </p>
    </checklist-step>
    <checklist-step stage="current">
      <template #title>{{ $t('steps[1].title') }}</template>
      <p v-if="status">
        {{ $t('steps[1].body[0].status') }}
      </p>
      <i18n-t v-else tag="p" keypath="steps[1].body[0].link.full">
        <template #upload>
          <router-link :to="formPath('draft')">{{ $t('steps[1].body[0].link.upload') }}</router-link>
        </template>
      </i18n-t>
    </checklist-step>
    <checklist-step v-if="attachments.get().size !== 0"
      :stage="attachments.missingCount === 0 ? 'complete' : 'current'">
      <template #title>{{ $t('steps[2].title') }}</template>
      <p>
        <i18n-t keypath="steps[2].body[0].full">
          <template #formAttachments>
            <router-link :to="formPath('draft/attachments')">{{ $t('steps[2].body[0].formAttachments') }}</router-link>
          </template>
        </i18n-t>
        <sentence-separator/>
        <doc-link to="central-forms/#forms-with-attachments">{{ $t('clickForInfo') }}</doc-link>
      </p>
    </checklist-step>
    <checklist-step
      :stage="formDraft.submissions !== 0 ? 'complete' : 'current'">
      <template #title>{{ $t('steps[3].title') }}</template>
      <p>
        <i18n-t keypath="steps[3].body[0].full">
          <template #test>
            <router-link :to="formPath('draft/testing')">{{ $t('steps[3].body[0].test') }}</router-link>
          </template>
        </i18n-t>
        <sentence-separator/>
        <doc-link to="central-forms/#working-with-form-drafts">{{ $t('clickForInfo') }}</doc-link>
      </p>
    </checklist-step>
    <checklist-step stage="current">
      <template #title>{{ $t('steps[4].title') }}</template>
      <p>
        <template v-if="status">{{ $t('steps[4].body[0].status') }}</template>
        <i18n-t v-else keypath="steps[4].body[0].link.full">
          <template #publish>
            <router-link :to="formPath('draft')">{{ $t('steps[4].body[0].link.publish') }}</router-link>
          </template>
        </i18n-t>
        <sentence-separator/>
        <doc-link to="central-forms/#working-with-form-drafts">{{ $t('clickForInfo') }}</doc-link>
      </p>
    </checklist-step>
  </div>
</template>

<script>
import ChecklistStep from '../checklist-step.vue';
import DocLink from '../doc-link.vue';
import SentenceSeparator from '../sentence-separator.vue';

import routes from '../../mixins/routes';
import { useRequestData } from '../../request-data';

export default {
  name: 'FormDraftChecklist',
  components: { ChecklistStep, DocLink, SentenceSeparator },
  mixins: [routes()],
  props: {
    // Indicates whether the current route path is .../draft.
    status: {
      type: Boolean,
      default: false
    }
  },
  setup() {
    // The component does not assume that this data will exist when the
    // component is created.
    const { form, attachments, resourceView, resourceStates } = useRequestData();
    const formDraft = resourceView('formDraft', (data) => data.get());
    const { dataExists } = resourceStates([form, formDraft, attachments]);
    return { form, formDraft, attachments, dataExists };
  }
};
</script>

<i18n lang="json5">
{
  "en": {
    "clickForInfo": "Click here to find out more.",
    "steps": [
      {
        // This is the title of a checklist item.
        "title": "Upload initial Form definition",
        "body": [
          "Great work!",
          "Your Form design has been loaded successfully."
        ]
      },
      {
        // This is the title of a checklist item.
        "title": "Upload revised Form definition (optional)",
        "body": [
          {
            // This refers to changes to the Form definition as opposed to just
            // the Attachments associated with the Form. The word "XLSForm" should not
            // be translated.
            "status": "If you have made changes to the Form itself, including question text or logic rules, now is the time to upload the new XML or XLSForm using the button to the right.",
            "link": {
              // This refers to changes to the Form definition as opposed to
              // just the Attachments associated with the Form. The word "XLSForm"
              // should not be translated.
              "full": "If you have made changes to the Form itself, including question text or logic rules, now is the time to {upload} the new XML or XLSForm.",
              "upload": "upload"
            }
          }
        ]
      },
      {
        // This is the title of a checklist item.
        "title": "Upload Form Attachments",
        "body": [
          {
            "full": "Your Form design references files that are needed in order to present your Form. You can upload new or updated copies of these for distribution under the {formAttachments} tab.",
            "formAttachments": "Form Attachments"
          }
        ]
      },
      {
        // This is the title of a checklist item.
        "title": "Test the Form by creating a Submission",
        "body": [
          {
            "full": "You can {test} a Form to be sure it works the way you expect. Test Submissions are not included in your final data.",
            "test": "test"
          }
        ]
      },
      {
        // This is the title of a checklist item.
        "title": "Publish the Draft",
        "body": [
          {
            "status": "When you are sure your Draft is ready and you wish to roll it out to your devices in the field, you can publish it using the button to the right.",
            "link": {
              "full": "When you are sure your Draft is ready and you wish to roll it out to your devices in the field, you can {publish} it.",
              "publish": "publish"
            }
          }
        ]
      }
    ]
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "clickForInfo": "Klikn??te sem a dozv??te se v??ce",
    "steps": [
      {
        "title": "Nahrajte po????te??n?? definici formul????e",
        "body": [
          "Skv??l?? pr??ce!",
          "N??vrh formul????e byl ??sp????n?? na??ten."
        ]
      },
      {
        "title": "Nahr??t revidovanou definici formul????e (voliteln??)",
        "body": [
          {
            "status": "Pokud jste provedli zm??ny v samotn??m formul????i, v??etn?? textu ot??zek nebo logick??ch pravidel, je nyn?? ??as nahr??t nov?? XML nebo XLSForm pomoc?? tla????tka napravo.",
            "link": {
              "full": "Pokud jste provedli zm??ny v samotn??m formul????i, v??etn?? textu ot??zek nebo logick??ch pravidel, je nyn?? ??as na {upload} nov??ho XML nebo XLSForm.",
              "upload": "nahr??t"
            }
          }
        ]
      },
      {
        "title": "Nahr??t p????lohy formul????e"
      },
      {
        "title": "Otestujte formul???? vytvo??en??m pod??n??",
        "body": [
          {
            "full": "M????ete {test} formul???? abyste se ujistili, ??e funguje tak, jak o??ek??v??te. Testovac?? odesl??n?? nejsou sou????st?? va??ich kone??n??ch ??daj??.",
            "test": "otestovat"
          }
        ]
      },
      {
        "title": "Publikovat koncept",
        "body": [
          {
            "status": "Pokud jste si jisti, ??e je koncept p??ipraven a chcete jej uv??st do provozu v ter??nu, m????ete jej publikovat pomoc?? tla????tka napravo.",
            "link": {
              "full": "Pokud jste si jisti, ??e je koncept p??ipraven a chcete jej uv??st do provozu v ter??nu, m????ete ho {publish}.",
              "publish": "publikovat"
            }
          }
        ]
      }
    ]
  },
  "de": {
    "clickForInfo": "Klicken Sie hier, um mehr zu erfahren.",
    "steps": [
      {
        "title": "Formulardefinition hochladen",
        "body": [
          "Gute Arbeit!",
          "Ihr Formular wurde erfolgreich hochgeladen."
        ]
      },
      {
        "title": "Ge??ndert Formulardefinition hochladen (optional)",
        "body": [
          {
            "status": "Wenn Sie ??nderungen am Formular selbst vorgenommen haben, etwa am Fragetext oder an den Logikregeln, sollten Sie jetzt das neue XML- oder XLSFormular mit der Schaltfl??che auf der rechten Seite hochladen.",
            "link": {
              "full": "Wenn Sie ??nderungen am Formular selbst vorgenommen haben, etwa am Fragetext oder an den Logikregeln, sollten Sie jetzt das neue Formular {upload}.",
              "upload": "hochladen"
            }
          }
        ]
      },
      {},
      {
        "title": "Teste das Formular indem du eine Einreichung erstellst.",
        "body": [
          {
            "full": "Sie k??nnen ein Formular {test}, um sicher zu sein, dass es wie erwartet funktioniert. Test-??bermittlungen sind in den endg??ltigen Daten nicht enthalten.",
            "test": "testen"
          }
        ]
      },
      {
        "title": "Den Entwurf ver??ffentlichen",
        "body": [
          {
            "status": "Wenn Sie sicher sind, dass Ihr Entwurf fertig ist und Sie ihn f??r den Feldeinsatz bereitstellen wollen, k??nnen Sie ihn mit der Schaltfl??che rechts ver??ffentlichen.",
            "link": {
              "full": "Wenn Sie sicher sind, dass Ihr Entwurf fertig ist und Sie ihn f??r den Feldeinsatz bereitstellen wollen, k??nnen sie ihn {publish}.",
              "publish": "ver??ffentlichen"
            }
          }
        ]
      }
    ]
  },
  "es": {
    "clickForInfo": "Haga clic aqu?? para descubrir m??s.",
    "steps": [
      {
        "title": "Subir definici??n de formulario inicial",
        "body": [
          "Buen trabajo.",
          "Su dise??o de formulario se ha cargado correctamente."
        ]
      },
      {
        "title": "Subir definici??n de formulario revisada (opcional)",
        "body": [
          {
            "status": "Si ha realizado cambios en el formulario en s??, incluido el texto de la pregunta o las reglas l??gicas, ahora es el momento de cargar el nuevo XML o XLSForm utilizando el bot??n a la derecha.",
            "link": {
              "full": "Si ha realizado cambios en el formulario en s??, incluido el texto de la pregunta o las reglas l??gicas, ahora es el momento de {upload} el nuevo XML o XLSForm.",
              "upload": "subir"
            }
          }
        ]
      },
      {},
      {
        "title": "Pruebe el formulario creando un env??o",
        "body": [
          {
            "full": "Puede {test} un formulario para asegurarse de que funciona como usted espera. Los env??os de prueba no son incluidos en sus datos finales.",
            "test": "probar"
          }
        ]
      },
      {
        "title": "Publicar el borrador",
        "body": [
          {
            "status": "Cuando est?? seguro de que su borrador est?? listo y desea extenderlo a sus dispositivos en el campo, puede publicarlo usando el bot??n a la derecha.",
            "link": {
              "full": "Cuando est?? seguro de que su borrador est?? listo y desea extenderlo a sus dispositivos en el campo, puede {publish}.",
              "publish": "publicarlo"
            }
          }
        ]
      }
    ]
  },
  "fr": {
    "clickForInfo": "Cliquez ici pour en savoir plus.",
    "steps": [
      {
        "title": "T??l??verser la d??finition initiale du formulaire",
        "body": [
          "Beau travail !",
          "Votre design de formulaire a ??t?? t??l??vers?? avec succ??s."
        ]
      },
      {
        "title": "T??l??verser une r??vision de la d??finition du formulaire (optionnel)",
        "body": [
          {
            "status": "Si vous avez apport?? des changements au formulaire lui-m??me, y compris au texte des questions ou au r??gles logiques, il est temps de t??l??verser le nouveau XML ou XLSForm en utilisant le bouton de droite.",
            "link": {
              "full": "Si vous avez apport?? des changements au formulaire lui-m??me, y compris au texte des questions ou au r??gles logiques, il est temps de {upload} le nouveau XML ou XLSForm.",
              "upload": "t??l??verser"
            }
          }
        ]
      },
      {
        "title": "Envoyer les fichiers joints du formulaire",
        "body": [
          {
            "full": "Votre d??finition de formulaire r??f??rence des fichiers n??cessaires ?? la pr??sentation de votre formulaire. Vous pouvez envoyer des copies nouvelles ou mises ?? jour de ces fichiers sous l'onglet {formAttachments}.",
            "formAttachments": "Fichiers joints de formulaire"
          }
        ]
      },
      {
        "title": "Testez le formulaire en cr??ant une soumission",
        "body": [
          {
            "full": "Vous pouvez {test} un formulaire pour v??rifier qu'il fonctionne comme pr??vu. Les soumissions de test ne seront pas incluses dans vos vraies donn??es.",
            "test": "tester"
          }
        ]
      },
      {
        "title": "Publier l'??bauche",
        "body": [
          {
            "status": "Quand vous ??tes s??r que votre ??bauche est pr??te et que vous souhaitez la d??ployer sur vos appareils sur le terrain, vous pouvez la publier en utilisant le bouton de droite.",
            "link": {
              "full": "Quand vous ??tes s??r que votre ??bauche est pr??te et que vous souhaitez la d??ployer sur vos appareils sur le terrain, vous pouvez la {publish}.",
              "publish": "publier"
            }
          }
        ]
      }
    ]
  },
  "id": {
    "clickForInfo": "Klik di sini untuk mencari tahu lebih.",
    "steps": [
      {
        "title": "Unggah definisi formulir awal",
        "body": [
          "Kerja bagus!",
          "Desain formulir Anda telah berhasil dimuat."
        ]
      },
      {
        "title": "Unggah definisi formulir yang sudah direvisi (opsional)",
        "body": [
          {
            "status": "Apabila Anda sudah membuat perubahan terhadap formulir, termasuk teks pertanyaan atau peraturan logis, sekarang adalah waktunya mengunggah XML baru atau XLSForm menggunakan tombol yang ada di sebelah kanan.",
            "link": {
              "full": "Apabila Anda sudah membuat perubahan terhadap formulir, termasuk teks pertanyaan atau peraturan logis, sekarang adalah waktunya {upload} XML baru atau XLSForm.",
              "upload": "Unggah"
            }
          }
        ]
      },
      {},
      {
        "body": [
          {
            "full": "Anda bisa {test} formulir untuk memastikan kesesuaiannya dengan yang Anda ekspektasikan. Kiriman data dari hasil tes tidak akan dimasukkan ke dalam data akhir Anda.",
            "test": "mengetes"
          }
        ]
      },
      {
        "title": "Terbitkan Draf",
        "body": [
          {
            "status": "Ketika Anda yakin Draf sudah siap, Anda bisa menerbitkannya menggunakan tombol yang ada di sebelah kanan.",
            "link": {
              "full": "Ketika Anda yakin Draf sudah siap, Anda bisa {publish} menggunakan tombol yang ada di sebelah kanan.",
              "publish": "menerbitkannya"
            }
          }
        ]
      }
    ]
  },
  "it": {
    "clickForInfo": "Clicca qui per ulteriori informazioni",
    "steps": [
      {
        "title": "Carica la definizione del formulario iniziale",
        "body": [
          "Ben fatto!",
          "Il design del tuo formulario ?? stato caricato con successo."
        ]
      },
      {
        "title": "Carica la definizione del formulario revisionata (opzionale)",
        "body": [
          {
            "status": "Se hai apportato modifiche al formulario stesso, incluso il testo della domanda o le regole logiche, ora ?? il momento di caricare il nuovo XML o XLSForm utilizzando il pulsante a destra.",
            "link": {
              "full": "Se hai apportato modifiche al formulario stesso, incluso il testo della domanda o le regole logiche, ora ?? il momento di {upload} il nuovo XML o XLSForm.",
              "upload": "carica"
            }
          }
        ]
      },
      {},
      {
        "title": "Testa il formulario creando un invio",
        "body": [
          {
            "full": "Puoi {test} un formulario per assicurarti che funzioni come previsto. Gli invii di prova non sono inclusi nei dati finali.",
            "test": "test"
          }
        ]
      },
      {
        "title": "Pubblica bozza",
        "body": [
          {
            "status": "Quando sei sicuro che la tua bozza ?? pronta e desideri distribuirla sui tuoi dispositivi sul campo, puoi pubblicarla utilizzando il pulsante a destra.",
            "link": {
              "full": "Quando sei sicuro che la tua bozza ?? pronta e desideri distribuirla sui tuoi dispositivi sul campo, puoi {publish}.",
              "publish": "pubblica"
            }
          }
        ]
      }
    ]
  },
  "ja": {
    "clickForInfo": "???????????????????????????????????????????????????????????????????????????",
    "steps": [
      {
        "title": "???????????????????????????????????????????????????????????????",
        "body": [
          "????????????????????????",
          "???????????????????????????????????????????????????"
        ]
      },
      {
        "title": "???????????????????????????????????????????????????????????????",
        "body": [
          {
            "status": "???????????????????????????????????????????????????????????????????????????????????????????????????????????????XML?????????XLSForm???????????????????????????????????????",
            "link": {
              "full": "????????????????????????????????????????????????????????????????????????????????????XML?????????XLSForm???{upload}??????????????????????????????????????????",
              "upload": "??????????????????"
            }
          }
        ]
      },
      {},
      {
        "title": "?????????????????????????????????????????????????????????????????????",
        "body": [
          {
            "full": "????????????????????????????????????????????????{test}?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????",
            "test": "?????????"
          }
        ]
      },
      {
        "title": "??????????????????",
        "body": [
          {
            "status": "????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????",
            "link": {
              "full": "????????????????????????????????????????????????????????????????????????????????????????????????????????????{publish}???????????????",
              "publish": "??????"
            }
          }
        ]
      }
    ]
  },
  "sw": {
    "clickForInfo": "Bonyeza hapa kujua zaidi",
    "steps": [
      {
        "title": "Pakia ufafanuzi wa awali wa Fomu",
        "body": [
          "Kazi nzuri!",
          "Muundo wako wa Fomu umepakiwa."
        ]
      },
      {
        "title": "Pakia ufafanuzi wa Fomu uliorekebishwa (si lazima)",
        "body": [
          {
            "status": "Ikiwa umefanya mabadiliko kwenye Fomu yenyewe, ikijumuisha maandishi ya swali au sheria za mantiki, sasa ni wakati wa kupakia XML au XLSForm mpya kwa kutumia kitufe kilicho kulia.",
            "link": {
              "full": "Ikiwa umefanya mabadiliko kwenye Fomu yenyewe, ikijumuisha maandishi ya swali au sheria za mantiki, sasa ni wakati wa {upload} XML au XLSForm mpya.",
              "upload": "pakia"
            }
          }
        ]
      },
      {},
      {
        "title": "Jaribu Fomu kwa kuunda Wasilisho",
        "body": [
          {
            "full": "Unaweza {test} Fomu ili kuhakikisha kuwa inafanya kazi jinsi unavyotarajia. Mawasilisho ya Jaribio hayajumuishwi katika data yako ya mwisho.",
            "test": "Jaribia"
          }
        ]
      },
      {
        "title": "Chapisha Rasimu",
        "body": [
          {
            "status": "Unapokuwa na uhakika kuwa Rasimu yako iko tayari na ungependa kuisambaza kwa vifaa vyako kwenye uga, unaweza kuichapisha kwa kutumia kitufe kilicho kulia.",
            "link": {
              "full": "Unapokuwa na uhakika kuwa Rasimu yako iko tayari na ungependa kuisambaza kwa vifaa vyako kwenye uga, unaweza {publish}.",
              "publish": "Chapisha"
            }
          }
        ]
      }
    ]
  }
}
</i18n>
