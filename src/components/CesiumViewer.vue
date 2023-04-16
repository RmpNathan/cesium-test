<template>
    <div id="cesiumContainer" style="height: 100vh"></div>
</template>
<script>
import * as Cesium from "cesium";
import monfichier from "../assets/geojson/tracks.geojson";

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
        const positions = monfichier.features[0].geometry.coordinates[0].map((coord) => {
            return Cesium.Cartographic.fromDegrees(
                coord[0],
                coord[1]
            );
        });
        const promise = Cesium.sampleTerrainMostDetailed(
            viewer.terrainProvider,
            positions
        );
        promise.then((updatedPositions) => {
            this.polylinePositions = updatedPositions.map((position) => {
                return Cesium.Cartesian3.fromRadians(
                    position.longitude,
                    position.latitude,
                    position.height
                );
            });

            const polyline = new Cesium.Entity({
                polyline: {
                    positions: this.polylinePositions,
                    width: 3,
                    material: Cesium.Color.RED,
                    clampToGround: true,
                },
            });
            console.log('polyline : ', polyline)
            viewer.entities.add(polyline);
            this.polylineEntity = polyline;
        });

        viewer.camera.flyTo({
            destination: Cesium.Cartesian3.fromDegrees(
                monfichier.features[0].geometry.coordinates[0][0][0],
                monfichier.features[0].geometry.coordinates[0][0][1],
                5000.0
            ),
            complete: () => {
                const bluePoint = new Cesium.Entity({
                    position: this.polylinePositions[0],
                    point: {
                        color: Cesium.Color.BLUE,
                        pixelSize: 10,
                    },
                });
                viewer.entities.add(bluePoint);
                this.bluePointEntity = bluePoint;
            }
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
