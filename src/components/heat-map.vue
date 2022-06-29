<template>
  <div class="heat-map">
    <h1>Change Heat Map</h1>
    <div id="calender">
      <div id="yearly-div"></div>
      <div id="monthly-div"></div>
      <div id="grid"></div>
    </div>
  </div>

  <view-changes :details="details" :Selecteddate="Selecteddate" />
</template>

<script lang="ts">
import {
  createElementBlock,
  defineComponent,
  provide,
  reactive,
  ref,
} from "vue";
import ViewChanges from "./view-changes.vue";
import Details from "../domains/DateDetails";
import Change from "../domains/changes";
import axios from "axios";
export default defineComponent({
  mounted() {
    this.fetchData();
  },
  components: { ViewChanges },

  setup() {
    var week = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];
    let monthNames = [
      "Jan",
      "Feb",
      "Mar",
      "Apr",
      "May",
      "Jun",
      "Jul",
      "Aug",
      "Sep",
      "Oct",
      "Nov",
      "Dec",
    ];
    var boxSize = 15;

    let changes = reactive({
      startDate: "",
      endDate: "",
      counts: {},
    }) as Change;

    let Selecteddate = ref("");

    let details = ref<Details[]>([]);

    let fetchData = async () => {
      await axios
        .get("https://tempheatmap.free.beeceptor.com/all")
        .then((response) => {
          changes.startDate = response.data.startDate;
          changes.endDate = response.data.endDate;
          changes.counts = response.data.counts;
          //console.log(changes.startDate);
        })
        .catch((err) => {
          console.log("There was an error", err);
        });

      displayHeatMap();
    };

    //FUNTION TO CALL DATE SPECIFIC
    let getDateSpecific = async (dateString: string) => {
      let isDateSelected = false;
      console.log("called");

      await axios
        .get(`https://tempheatmap.free.beeceptor.com/${dateString}`)
        .then((response) => {
          console.log(response.data);
          details.value = response.data;
          Selecteddate.value = dateString;
        })
        .catch((err) => {
          console.log("There was an error", err);
        });
    };

    //FUNCTION TO DISPLAY HEATMAP
    function displayHeatMap() {
      const Dateformatter = new Intl.DateTimeFormat("en-CA");
      const Monthformatter = new Intl.DateTimeFormat("en-CA", {
        month: "short",
      });
      let startDate = new Date(changes.startDate) as Date;
      let endDate = new Date(changes.endDate) as Date;
      let counts = changes.counts as Record<string, number>;
      var keyDates = Object.keys(counts);

      //ARRAY OF YEARS
      var years = [] as number[];
      for (let i = startDate.getFullYear(); i <= endDate.getFullYear(); i++) {
        years.push(i);
      }

      //CALCULATING NUMBER OF MONTHS
      var months =
        endDate.getUTCMonth() -
        startDate.getUTCMonth() +
        12 * (endDate.getFullYear() - startDate.getFullYear());

      //FUNCTION TO CALCULATE WEEK OF MONTH FOR DATE
      let weekOfMonth = (date: Date) => {
        return Math.ceil((date.getDate() - 1 - date.getDay()) / 7);
      };

      //GET MONTH NAME
      let getMonthName = (monthNumber: number) => {
        return monthNames[monthNumber];
      };

      //GET YEAR INDEX
      let getYearIndex = (date: Date) => {
        return years.findIndex((currentYear, index) => {
          return currentYear === date.getFullYear();
        });
      };

      //GET BACKGROUND COLOR FOR NODE
      let colorOfNode = (date: Date): string => {
        let tempDate = Dateformatter.format(date) as string;
        let color = "";

        for (let i = 0; i < keyDates.length; i++) {
          if (keyDates[i] === tempDate) {
            let count = counts[tempDate];

            if (count > 1 && count <= 5) {
              color = "#FDA08B";
              break;
            } else if (count >= 6 && count <= 8) {
              color = "#FC8D72";
              break;
            } else if (count >= 9 && count <= 11) {
              color = "#FC7759";
              break;
            } else if (count >= 12) {
              color = "#FC6340";
              break;
            }
          } else {
            color = "#DCE0E5";
          }
        }

        return color;
      };

      //GET CHANGES
      let getChanges = (date: Date) => {
        let tempDate = Dateformatter.format(date) as string;
        let count = 0;
        for (let i = 0; i < keyDates.length; i++) {
          if (keyDates[i] === tempDate) {
            count = counts[tempDate];
            break;
          } else count = 0;
        }
        return count;
      };

      //FUNCTION TO RETURN DATE FORMAT
      let toDateFormat = (date: Date) => {
        const yyyy: string | number = date.getFullYear();
        let mm: string | number = date.getMonth() + 1;
        let dd: string | number = date.getDate();

        if (dd < 10) dd = "0" + dd;
        if (mm < 10) mm = "0" + mm;

        return dd + "-" + mm + "-" + yyyy;
      };

      //CREATING AND APPENDEING YEAR LABEL
      let yearlyDiv = document.querySelector("#yearly-div") as HTMLElement;

      years.map((currentYear, index) => {
        var yearContainer = document.createElement("div");
        yearContainer.className = "year-container";
        yearContainer.innerHTML = currentYear.toString();
        yearContainer.style.marginLeft = index * (boxSize * 83 + 15) + "px";

        yearlyDiv.append(yearContainer);
      });

      let calenderDiv = document.querySelector("#grid") as HTMLElement;
      let weekLabel = document.createElement("div");
      weekLabel.className = "weekLabel";

      //ATTACHING WEEK LABELS
      week.map((weekDay, index) => {
        let weekName = document.createElement("div");
        weekName.className = "week-label";
        weekName.style.marginTop = index * boxSize + 2 * index + "px";
        if (weekDay === "Mon" || weekDay === "Wed" || weekDay === "Fri")
          weekName.innerHTML += weekDay;
        weekLabel.append(weekName);
      });
      calenderDiv.append(weekLabel);

      //ATTACHING MONTH LABLES
      let monthlyDiv = document.getElementById("monthly-div") as HTMLElement;
      for (let i = startDate.getUTCMonth(); i < months; i++) {
        let monthlyLabel = document.createElement("div");
        monthlyLabel.className = "monthly-label";
        monthlyLabel.innerHTML = getMonthName(i);
        monthlyLabel.style.marginLeft = (i + 1) * (boxSize * 6 + 15) + "px";
        monthlyDiv.append(monthlyLabel);
      }

      //CREATING DATEWISE GRID
      let date = startDate;

      while (date <= endDate) {
        var ctr = document.createElement("div");
        ctr.className = "ctr";
        ctr.id = toDateFormat(date);

        ctr.style.marginTop =
          date.getDay() * boxSize + 3 * date.getDay() + "px";

        let dateYear = getYearIndex(date);

        ctr.style.marginLeft =
          dateYear * 1200 +
          (date.getMonth() + 1) * (boxSize * 6 + 15) +
          weekOfMonth(date) * boxSize +
          3 * weekOfMonth(date) +
          "px";

        ctr.style.backgroundColor = colorOfNode(date);

        ctr.setAttribute("data-bs-toggle", "tooltip");
        ctr.setAttribute("data-bs-placement", "Tooltip on top");
        let toolTipStr =
          getChanges(date) +
          " changes on " +
          getMonthName(date.getMonth()) +
          " " +
          date.getDate() +
          "," +
          date.getFullYear();
        ctr.setAttribute("title", toolTipStr);

        ctr.addEventListener("click", (e) => {
          let div = e.target as HTMLElement;
          getDateSpecific(div.id);
          console.log(div.id);
        });
        ctr.innerHTML = date.getDate().toString();
        calenderDiv.appendChild(ctr);
        date.setDate(date.getDate() + 1);
      }
    }
    // provide("details", details);
    return {
      displayHeatMap,
      fetchData,
      details,
      Selecteddate,
    };
  },
});
</script>
<style>
.heat-map {
  -webkit-box-shadow: 0px 0px 25px 10px rgba(227, 227, 227, 1);
  -moz-box-shadow: 0px 0px 25px 10px rgba(227, 227, 227, 1);
  box-shadow: 0px 0px 25px 10px rgba(227, 227, 227, 1);
  width: 80%;
  padding: 20px;
  background-color: white;
  margin-top: 70px;
  color: #262658;
  font-size: 19px !important;
}
.heat-map h1 {
  font-weight: 800;
}
#calender {
  display: flex;
  flex-direction: row;
  font-size: 16px;
  overflow: hidden;
}

.ctr {
  width: 15px;
  height: 15px;
  position: absolute;
  font-size: 9px;
  cursor: pointer;
}

.week-label {
  position: absolute;
  margin-left: 60px;
}
.monthly-label {
  position: absolute;
  margin-top: 30px;
}
#grid {
  margin-top: 55px;
  height: 15vh;
}

#yearly-div {
  position: absolute;
  margin-left: 90px;
}
.year-container {
  position: absolute;
}
</style>
