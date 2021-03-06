<template>
    <div
        draggable="true"
        @dragstart="setDragImage"
        @click="scrollToCard"
        @contextmenu.prevent="removePiece">
        <img
            :id="charInfo.className + 'Image'"
            ref="GridPieceImg" 
            src=""
            alt="GP">
    </div>
</template>

<script>
import { mapActions, mapGetters, mapMutations} from 'vuex';
import characterCardMixin from '../../mixins/characterCardMixin';
import gridMixin from '../../mixins/gridMixin';

export default {
    name: "GridPiece",
    props: ['charInfo', 'position'],
    computed: mapGetters(['currentPreset', 'currentCharacter']),
    mixins: [characterCardMixin, gridMixin],
    methods: {
        ...mapActions(['removeGridPiece']),
        ...mapMutations(['setCurrentCharacter']),
        fillCoordinateColors(style){
            let {rowIndex, cellIndex} = this.position;
            let position = (rowIndex * 22) + cellIndex;
            let {className, coordinates, archetype} = this.currentPreset[position];
            let legionrow = [...document.getElementsByClassName('LegionRow')]; 
            coordinates.forEach(coord => {
                let {x, y} = coord;
                x += cellIndex;
                y += rowIndex;
                if(this.isWithinGrid(x, y)){
                    legionrow[y].children[x].style.cssText = style;
                }
            }) 
        },
        removePiece(e){
            let charInfo = {...this.charInfo, position: this.position};
            this.setArchetypes({...charInfo, append: false});
            this.removeGridPiece(charInfo);
            this.removeAllHighlights('CharacterCard');
        },
        setDragImage(e){
            this.deactivateCurrentCard(this);
            this.unhighlightCard(this);
            let {className} = this.charInfo;
            let piece = document.getElementById(className + 'Piece');
            e.dataTransfer.setDragImage(piece, 80, 80);
            this.setCurrentChar();
        },
        isTabletView(){
            let viewportWidth = Math.max(document.documentElement.clientWidth, window.innerWidth || 0);
            return viewportWidth <= 900;
        },
        scrollToCard(e){
            this.deactivateCurrentCard(this);
            let {className} = this.charInfo;
            let piece = document.getElementById(className + 'Piece');
            document.getElementById('Home').scrolling = 'no';
            let charCard = document.getElementById(className + 'Card');
            if(this.isTabletView()) {
                charCard.parentNode.scrollLeft = charCard.offsetLeft;
            }
            else {
                charCard.parentNode.scrollTop = charCard.offsetTop;
            }
            this.highlightCard(className);
            this.setCurrentChar();
        },
        setCurrentChar(){
            let {charInfo, position} = this;
            charInfo = {...charInfo, position};
            this.setCurrentCharacter(charInfo)
        }
    },
    mounted(){
        let icon = this.getImage(this.charInfo.className);
        this.$refs.GridPieceImg.src = icon;
        this.setArchetypes({
            ...this.charInfo,
            position: this.position,
            append: true
        })
    },
    updated(){
        this.setArchetypes({
            ...this.charInfo,
            position: this.position,
            append: true
        })
    }
}
</script>

<style scoped lang="scss">
    div {
        width: 100%;
        height: 100%;
        position: relative;
    }
    img {
        width: 100%;
        position: absolute;
        top: 0;
        left: 0;
        object-fit: cover;
        cursor: url('../../assets/ms-cursor.png'), auto;
    }

</style>