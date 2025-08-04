<script setup>
import { onMounted, reactive } from 'vue';

const state = reactive({
    address: [
        '충청남도 아산시 용화로 87-1',
        '대전광역시 유성구 봉산로16번길 51',
        '경상북도 문경시 산북면 내화1길',
        '경기도 고양시 덕양구 원당로392번길 188',
        '경기도 김포시 양촌읍 김포대로1759번길 51',
        '경상남도 남해군 창선면 서부로1066번길 22'
    ]
});

const myAddress = '대구 중구 중앙대로 394';

let map = null;
let geocoder = null;

// 카카오맵 API 스크립트를 불러오는 함수
// 네이버 지도와 달리 카카오맵은 스크립트를 불러온 뒤 동적으로 삽입해야 한다고 함
const loadKakaoMapScript = () => {
    return new Promise((resolve, reject) => {
        if (window.kakao && window.kakao.maps) {
            resolve(); // 이미 로드됨
            return;
        }

        const script = document.createElement('script');
        script.src = `https://dapi.kakao.com/v2/maps/sdk.js?appkey=${import.meta.env.VITE_KAKAO_MAP_CLIENT_ID}&autoload=false&libraries=services`; // autoload=false는 직접 init 하겠다는 뜻
        script.onload = () => {
            window.kakao.maps.load(resolve); // SDK 내부 로딩이 완료되면 실행
        };
        script.onerror = reject;

        document.head.appendChild(script);
    });
};

// 주소를 검색하여 지도에 마커를 표시하는 함수
const markingAddress = address => {
    if (!address) {
        return;
    }

    geocoder.addressSearch(address, (result, status) => {
        if (status === kakao.maps.services.Status.OK) {
            const coords = new kakao.maps.LatLng(result[0].y, result[0].x);

            map.setCenter(coords);

            if (marker) {
                marker.setPosition(coords);
            } else {
                marker = new kakao.maps.Marker({ map, position: coords });
            }
        } else {
            console.warn('주소 검색 결과가 없습니다.');
        }
    });
};

onMounted(async () => {
    await loadKakaoMapScript(); // 로드 기다림

    const container = document.querySelector('#map'); // 지도를 표시할 div
    const options = {
        center: new kakao.maps.LatLng(37.5665, 126.9780), // 서울 좌표
        level: 3,
    };

    map = new kakao.maps.Map(container, options);

    geocoder = new kakao.maps.services.Geocoder();

    markingAddress(state.address[0]);
});
</script>

<template>
    <div id="map"></div>
</template>

<style scoped>
#map {
    width: 400px;
    height: 400px;
}
</style>