<template>
    <div id="cesiumContainer" style="height: 100vh"></div>
</template>
<script>
import * as Cesium from "cesium";
// import monfichier from "../assets/geojson/tracks.geojson";

export default {
    name: "CesiumViewer",
    data() {
        return {
            viewer: null,
            bluePointEntity: null,
            polylineEntity: null,
            currentIndex: 0,
            speed: 1, // Vitesse de déplacement du point en m/s
            timer: null, // Timer pour la mise à jour de la position du point
            polylinePositions: []
        };
    },
    mounted() {
        const viewer = new Cesium.Viewer("cesiumContainer", {
            animation: false,
            timeline: false,
            geocoder: false,
            baseLayerPicker: false,
            sceneModePicker: false,
            fullscreenButton: false,
            homeButton: false,
            navigationHelpButton: false,
            infoBox: false,
            selectionIndicator: false,
            shadows: false,
            terrainProvider: Cesium.createWorldTerrain(),
        });

        // viewer.camera.flyTo({
        //     destination: Cesium.Cartesian3.fromDegrees(
        //         monfichier.features[0].geometry.coordinates[0][0][0],
        //         monfichier.features[0].geometry.coordinates[0][0][1],
        //         5000.0
        //     ),
        // });

        viewer.dataSources.add(Cesium.GpxDataSource.load(
            "/geojson/tm.gpx",
            {
                clampToGround: true,
            }
        )).then(function (dataSource) {
            viewer.flyTo(dataSource.entities);
            // const activeDataSource = dataSource;

            viewer.clock.onTick.addEventListener(function () {
                // This is admittedly not ideal, however, currently, this seems necessary to me
                // since we do not read entity IDs from GPX and they are generated at runtime.
                // const entity = activeDataSource.entities._entities._array[0];
                Cesium.Cartesian3.fromDegrees(-72.0, 40.0);
                // const heading = Cesium.Math.toRadians(50.0);
                // const pitch = Cesium.Math.toRadians(-20.0);
                // const range = 5000.0;
                // viewer.camera.lookAt(entity.position.getValue(clock.currentTime), new Cesium.HeadingPitchRange(heading, pitch, range));
            });
        });

        this.viewer = viewer;
    },
    beforeUnmount() {
        if (this.viewer && !this.viewer.isDestroyed()) {
            clearInterval(this.timer);
            this.viewer.destroy();
        }
    },
};
</script>

<style>
@import "~cesium/Build/Cesium/Widgets/widgets.css";
@import "~cesium/Build/Cesium/Widgets/CesiumWidget/CesiumWidget.css";

#cesiumContainer {
    width: 100%;
    margin: 0;
    padding: 0;
}

html,
body {
    width: 100%;
    height: 100%;
    margin: 0;
    padding: 0;
    overflow: hidden;
}
</style>
