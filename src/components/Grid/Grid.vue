<template>
    <div id="LegionGrid"
        assigning="true">
        <div 
            class="LegionRow"
            v-for="(row, rowIndex) in rows"
            :key="rowIndex"
            :rowIndex="rowIndex">
            <div 
                class="LegionCell"
                v-for="(cell, cellIndex) in cells"
                :key="cellIndex"
                :row="rowIndex"
                :cell="cellIndex"
                :archetypeList="''"
                @dragover.prevent="highlightCells"
                @dragleave="highlightCells"
                @drop="insertPiece(rowIndex, cellIndex, $event)">
                    <GridPiece 
                        v-if="isInPreset(rowIndex, cellIndex)" 
                        :charInfo="isInPreset(rowIndex, cellIndex)"
                        :position="{rowIndex, cellIndex}"
                    />
            </div>
        </div>
    </div>
</template>

<script>
import GridPiece from './GridPiece';
import { mapActions, mapGetters } from 'vuex';
import characterCardMixin from '../../mixins/characterCardMixin';
import gridMixin from '../../mixins/gridMixin';

export default {
    name: "Grid",
    components: {
        GridPiece
    },
    data(){
        return {
            rows: [...new Array(20)].map((x, i) => i),
            cells: [...new Array(22)].map((x, i) => i)
        }
    },
    computed: {
        ...mapGetters(['currentCharacter', 'currentPreset'])
    },
    mixins: [characterCardMixin, gridMixin],
    methods:{
        ...mapActions(['insertIntoPreset']),
        isInPreset(rowIndex, cellIndex){
            return this.currentPreset[rowIndex * 22 + cellIndex];
        },
        highlightCells(e){
            let dragover = (e.type === 'dragover');
            if(this.isNotOverlappingPieces(e)){
                let {coordinates} = this.currentCharacter;
                let {row, cell} = e.target.attributes;
                row = parseInt(row.value);
                cell = parseInt(cell.value);
                this.updateHighlightAttribute({dragover, row, cell});
                let coordsInGrid = this.coordinatesInGrid({row, cell, coordinates});
                coordsInGrid.forEach(coord => {
                    let {x, y} = coord;
                    x += cell;
                    y += row;
                    this.updateHighlightAttribute({
                        dragover, 
                        row: y, 
                        cell: x
                    })
                })
            }
        },
        isNotOverlappingPieces(e){
            return this.currentCharacter && e.target.nodeName !== "IMG";
        },
        updateHighlightAttribute(eventInfo){
            let {dragover, row, cell} = eventInfo;
            let legioncell = document.getElementsByClassName('LegionRow')[row].children[cell];
            if(dragover){
                legioncell.setAttribute('highlighted', true);
            }
            else {
                legioncell.removeAttribute('highlighted');
            }
        },
        insertPiece(rowIndex, cellIndex, e){
            let position = (rowIndex * 22) + cellIndex;
            if(this.isPositionEmpty(position)) {
                this.setCellArchetypes(position);
                this.insertIntoPreset(position);
                this.removeAllHighlights('LegionCell');
                this.deactivateCurrentCard(this);
                this.unhighlightCard(this);
            }
        },
        isPositionEmpty(position){
            return this.currentPreset[position] ? false : true;
        },
        setCellArchetypes(position){
            let {currentCharacter} = this;
            if(currentCharacter.position) {
                this.setArchetypes({
                    ...currentCharacter,
                    append: false
                })
            }
        }
    }
}
</script>

<style scoped lang="scss">
    @import '../../variables.scss';
    @import '../../mixins.scss';

    #LegionGrid {
        @include grid-layout;
        &[assigning="true"]{
            background: none;
        }
    }

    .LegionRow {
        display: flex;
        justify-content: center;
    }

    .LegionCell {
        @include cell-layout;
        @include archetype-colors;
        &[archetype="Overlap"]{ 
            display: flex;
            justify-content: center;
            align-items: center;
            border: 1px white solid;
            background: {
                image: url('../../assets/Overlap.svg');
                size: cover;
            }
        }
        &[highlighted="true"]{ border: 3px dashed yellow; }
    }

    @include desktop-large-grid-size (
        '#LegionGrid',
        '.LegionCell'
    );

    @include tablet-grid-size (
        '#LegionGrid',
        '.LegionCell'
    );
    
</style>