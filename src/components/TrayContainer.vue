<template>
  <div class="text-center">
    <div class="text-h2 q-my-xl">Tray</div>
    <q-btn
      push
      color="primary"
      class="qt-mt-xl"
      label="Back to Home"
      icon="home"
      @click="$router.replace('/')"
    />
  </div>

  <div class="text-center">
    <div class="text-h4 q-mt-xl">Tray Contents</div>
    <q-avatar class="q-mt-md" size="xl" square>
      <img :src="`tray.png`" />
    </q-avatar>
    <div class="text-h6 q-mt-lg text-positive">{{ state.status }}</div>
  </div>
  <div
    v-if="state.tray.length > 0"
    style="width: 90vw; margin-left: 5vw; margin-top: 1vh"
  >
    <div>
      <q-item style="bottom: -2vh">
        <q-item-section class="col-2 q-ml-sm text-h6 text-primary"
          >QTY</q-item-section
        >
        <q-item-section class="q-ml-sm text-h6 text-primary"
          >DESCRIPTION</q-item-section
        >
      </q-item>
      <q-scroll-area style="height: 40vh">
        <q-card class="q-ma-md">
          <q-list separator>
            <q-item v-for="item in state.tray" :key="item.id" clickable>
              <q-item-section class="col-2">
                {{ item.qty }}
              </q-item-section>
              <q-item-section class="text-left">
                {{ item.item.description }}
              </q-item-section>
            </q-item>
          </q-list>
        </q-card>
      </q-scroll-area>
    </div>
    <div row>
      <div class="text-center q-mb-xl">
        <q-btn
          glossy
          color="negative"
          icon="delete"
          class="q-mt-xl"
          @click="clearTray()"
          label="Clear Tray"
        />
      </div>
    </div>
    <div class="text-h6 text-center text-primary">Nutritional Totals</div>
    <table style="border: solid; margin: 2em">
      <tr>
        <td style="width: 20%; font-weight: bold">Protein</td>
        <td style="width: 10%; text-align: right; padding-right: 3vw">
          {{ state.proTot }}
        </td>
        <td style="width: 20%; font-weight: bold">Calories</td>
        <td style="width: 10%; text-align: right; padding-right: 3vw">
          {{ state.calTot }}
        </td>
        <td style="width: 20%; font-weight: bold">Carbs.</td>
        <td style="width: 10%; text-align: right; padding-right: 3vw">
          {{ state.carbTot }}
        </td>
      </tr>
      <tr>
        <td style="width: 20%; font-weight: bold">Fibre</td>
        <td style="width: 10%; text-align: right; padding-right: 3vw">
          {{ state.fbrTot }}
        </td>
        <td style="width: 20%; font-weight: bold">Choles.</td>
        <td style="width: 10%; text-align: right; padding-right: 3vw">
          {{ state.cholTot }}
        </td>
        <td style="width: 20%; font-weight: bold">Salt</td>
        <td style="width: 10%; text-align: right; padding-right: 3vw">
          {{ state.saltTot }}
        </td>
      </tr>
      <tr>
        <td style="width: 20%; font-weight: bold">Fat</td>
        <td style="width: 10%; text-align: right; padding-right: 3vw">
          {{ state.fatTot }}
        </td>
        <td colspan="4">&nbsp;</td>
      </tr>
    </table>
  </div>
</template>
<script>
import { reactive, onMounted } from "vue";
import { fetcher } from "../utils/apiutil";
import { useRouter } from "vue-router";
export default {
  setup() {
    const router = useRouter();
    onMounted(() => {
      loadCategories();
    });

    let state = reactive({
      status: "",
      fbrTot: 0,
      calTot: 0,
      saltTot: 0,
      fatTot: 0,
      cholTot: 0,
      proTot: 0,
      carbTot: 0,
      tray: [],
    });

    onMounted(() => {
      if (sessionStorage.getItem("tray")) {
        state.tray = JSON.parse(sessionStorage.getItem("tray"));
        state.tray.forEach((trayItem) => {
          state.fbrTot += trayItem.item.fibre * trayItem.qty;
          state.calTot += trayItem.item.calories * trayItem.qty;
          state.saltTot += trayItem.item.salt * trayItem.qty;
          state.fatTot += trayItem.item.fat * trayItem.qty;
          state.cholTot += trayItem.item.cholesterol * trayItem.qty;
          state.proTot += trayItem.item.protein * trayItem.qty;
          state.carbTot += trayItem.item.carbs * trayItem.qty;
        });
      } else {
        state.tray = [];
      }
    });

    const loadCategories = async () => {
      try {
        state.status = "loading categories...";
        const payload = await fetcher(`Category`);
        if (payload.error === undefined) {
          state.categories = payload;
          state.status = `loaded ${state.categories.length} categories`;
        } else {
          state.status = payload.error;
        }
      } catch (err) {
        console.log(err);
        state.status = `Error has occured: ${err.message}`;
      }
    };

    const loadMenuitems = async () => {
      try {
        state.selectedCategory = state.categories.find(
          (category) => category.id === state.selectedCategoryId
        );
        state.status = `finding menuitems for category ${state.selectedCategory}...`;
        state.menuitems = await fetcher(
          `Menuitem/${state.selectedCategory.id}`
        );
        state.status = `loaded ${state.menuitems.length} menu items for ${state.selectedCategory.name}`;
      } catch (err) {
        console.log(err);
        state.status = `Error has occured: ${err.message}`;
      }
    };

    const selectMenuItem = async (menuitemid) => {
      try {
        state.selectedMenuItem = state.menuitems.find(
          (item) => item.id === menuitemid
        );
        state.itemSelected = true;
        state.dialogStatus = "";
        if (sessionStorage.getItem("tray")) {
          state.tray = JSON.parse(sessionStorage.getItem("tray"));
        }
      } catch (err) {
        console.log(err);
        state.status = `Error has occured: ${err.message}`;
      }
    };

    const addToTray = () => {
      let index = -1;
      if (state.tray.length > 0) {
        index = state.tray.findIndex(
          (item) => item.id === state.selectedMenuItem.id
        );
      }

      if (state.qty > 0) {
        index === -1
          ? state.tray.push({
              id: state.selectedMenuItem.id,
              qty: state.qty,
              item: state.selectedMenuItem,
            })
          : (state.tray[index] = {
              id: state.selectedMenuItem.id,
              qty: state.qty,
              item: state.selectedMenuItem,
            });
        state.dialogStatus = `${state.qty} item(s) added`;
      } else {
        index === -1 ? null : state.tray.splice(index, 1);
        state.dialogStatus = `item(s) removed`;
      }
      sessionStorage.setItem("tray", JSON.stringify(state.tray));
      state.qty = 0;
    };

    const viewTray = () => {
      router.push("tray");
    };

    const clearTray = () => {
      sessionStorage.removeItem("tray");
      state.tray = [];
      state.status = "tray cleared";
    };

    return {
      state,
      loadMenuitems,
      selectMenuItem,
      addToTray,
      viewTray,
      clearTray,
    };
  },
};
</script>
