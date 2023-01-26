<template>
  <div class="base">
    <ul class="g-snows" id="jsi-snows">
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
    </ul>
    <div id="overlay"></div>

    <div class="float-sm">
      <div class="fl-fl float-music">
        <i class="fa fa-music"></i>
        <a href="#" target="_blank">List Music!</a>
      </div>
    </div>

    <div class="content">
      <div class="lyrics"></div>
    </div>

    <div class="player">
      <div class="left"></div>
      <div class="right">
        <div class="top">
          <p class="songs">
            Centimeter
            <br />The Peggies (Kanojo - Okarishimasu OST SoundTrack Opening)
          </p>
        </div>
        <div class="bottom">
          <video
            controlslist="nodownload"
            controls=""
            _autoplay=""
            name="media"
            oncontextmenu="return false;"
          >
            <source id="music" type="audio/mpeg" />
          </video>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import "../assets/great.css";
import $ from "jquery";
import myMusic from "../assets/thepeggies.mp3";
import { onMounted, reactive } from "vue";

onMounted(() => {
  mounting();
  console.log(`the component is now mounted.`);
  document.getElementById("music").src = myMusic;
});

const state = reactive({
  count: 0,
  _data: "",
  currentLine: "",
});

async function setLyric() {
  await fetch("./lyric.json", {
    method: "GET",
  })
    .then((response) => response.json())
    .then((result) => {
      state._data = result;
    })
    .catch((error) => {
      console.error("Error:", error);
    });
}

function align() {
  let a = $(".highlighted").height();
  let c = $(".content").height();
  let d =
    $(".highlighted").offset().top - $(".highlighted").parent().offset().top;
  let e = d + a / 2 - c / 2;
  $(".content").animate(
    { scrollTop: e + "px" },
    { easing: "swing", duration: 250 }
  );
}

let lyricHeight = $(".lyrics").height();
$(window).on("resize", function () {
  if ($(".lyrics").height() != lyricHeight) {
    //Either width changes so that a line may take up or use less vertical space or the window height changes, 2 in 1
    lyricHeight = $(".lyrics").height();
    align();
  }
});

$(document).ready(function () {
  $("video").on("timeupdate", function () {
    let time = this.currentTime * 1000;
    let past = state._data["lyrics"].filter(function (item) {
      return item.time < time;
    });
    if (state._data["lyrics"][past.length] != state.currentLine) {
      state.currentLine = state._data["lyrics"][past.length];
      $(".lyrics div").removeClass("highlighted");
      $(`.lyrics div:nth-child(${past.length})`).addClass("highlighted"); //Text might take up more lines, do before realigning
      align();
    }
  });
});

function generate() {
  let html = "";
  for (let i = 0; i < state._data["lyrics"].length; i++) {
    html += "<div";
    if (i == 0) {
      html += ` class="highlighted"`;
      state.currentLine = 0;
    }
    if (state._data["lyrics"][i]["note"]) {
      html += ` note="${state._data["lyrics"][i]["note"]}"`;
    }
    html += ">";
    html +=
      state._data["lyrics"][i]["line"] == ""
        ? "•"
        : state._data["lyrics"][i]["line"];
    html += "</div>";
  }
  $(".lyrics").html(html);
  align();
}

async function mounting() {
  await setLyric();
  await generate();
}
</script>
