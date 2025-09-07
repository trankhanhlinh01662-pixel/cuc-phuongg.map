<!DOCTYPE html>
}
});


// 11) Geocoder (tìm trên OSM – hỗ trợ gõ từ khoá rộng)
L.Control.geocoder({ defaultMarkGeocode: false, placeholder: 'Tìm kiếm OSM…' })
.on('markgeocode', function(e) { map.fitBounds(e.geocode.bbox); })
.addTo(map);


// 12) Vẽ tuyến/điểm (ghi chú hiện trường)
const drawnItems = new L.FeatureGroup();
map.addLayer(drawnItems);
new L.Control.Draw({
position: 'topright',
draw: { polygon: true, polyline: true, rectangle: true, circle: false, marker: true, circlemarker: false },
edit: { featureGroup: drawnItems }
}).addTo(map);
map.on(L.Draw.Event.CREATED, function (e) { drawnItems.addLayer(e.layer); });


// 13) Tuyến đường gợi ý (Routing) dùng OSRM demo server
const routing = L.Routing.control({
waypoints: [ ],
routeWhileDragging: true,
geocoder: L.Control.Geocoder.nominatim(),
showAlternatives: true,
altLineOptions: { styles: [{opacity: 0.7}, {opacity: 0.7}] },
position: 'topright'
}).addTo(map);


// Click vào bản đồ để thêm waypoint (tối đa 2 cho đơn giản)
map.on('click', (e) => {
const wps = routing.getWaypoints().filter(w => w.latLng); // hiện có
if (wps.length >= 2) { routing.setWaypoints([e.latlng]); } // reset với điểm đầu mới
else { routing.spliceWaypoints(wps.length, 1, e.latlng); }
});


// 14) Đường mòn minh hoạ (GeoJSON đơn giản)
const trails = {
"type": "FeatureCollection",
"features": [
{"type":"Feature","properties":{"name":"Đường mòn Thung lũng"},"geometry":{"type":"LineString","coordinates":[[105.6005,20.3145],[105.5960,20.3100],[105.5905,20.3060],[105.5850,20.3015]]}},
{"type":"Feature","properties":{"name":"Lối vào Cây Chò"},"geometry":{"type":"LineString","coordinates":[[105.5997,20.3166],[105.5972,20.3140],[105.5950,20.3122],[105.5948,20.3190]]}}
]
};
function styleTrail() { return { color: '#22c55e', weight: 3 }; }
L.geoJSON(trails, { style: styleTrail, onEachFeature: (f, l) => l.bindPopup(f.properties.name) }).addTo(map);


// 15) Chú giải (Legend)
const legend = L.control({ position: 'bottomleft' });
legend.onAdd = function() {
const div = L.DomUtil.create('div', 'legend');
div.innerHTML = `
<h4>Chú giải</h4>
<div class="row"><span class="badge" style="background:${iconColors.facility}"></span> Dịch vụ/Trung tâm</div>
<div class="row"><span class="badge" style="background:${iconColors.cave}"></span> Hang động</div>
<div class="row"><span class="badge" style="background:${iconColors.tree}"></span> Cây cổ thụ</div>
<div class="row"><span class="badge" style="background:${iconColors.viewpoint}"></span> Cảnh quan</div>
<div class="row"><span class="badge" style="background:${iconColors.trailhead}"></span> Đầu đường mòn</div>
<div class="row"><span class="badge" style="background:#22c55e"></span> Đường mòn</div>
`;
return div;
};
legend.addTo(map);


// 16) Gợi ý: tải/sao lưu các ghi chú đã vẽ (xuất GeoJSON)
function exportDrawn() {
const gj = drawnItems.toGeoJSON();
const dataStr = 'data:text/json;charset=utf-8,' + encodeURIComponent(JSON.stringify(gj));
const a = document.createElement('a');
a.href = dataStr; a.download = 'ghi-chu-cuc-phuong.geojson'; a.click();
}


// Thêm phím tắt: Ctrl+S để xuất GeoJSON các ghi chú
document.addEventListener('keydown', (e) => {
if ((e.ctrlKey || e.metaKey) && e.key.toLowerCase() === 's') {
e.preventDefault(); exportDrawn();
}
});


</script>
</body>
</html>
