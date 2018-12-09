<template>
    <div class="ebook">
        <title-bar :ifTitleAndMenuShow="ifTitleAndMenuShow"></title-bar>
        <div class="read-wrapper">
            <div id="read"></div>
            <div class="mask">
                <div class="left" @click="prevPage"></div>
                <div class="center" @click="toggleTitleAndMenu"></div>
                <div class="right" @click="nextPage"></div>
            </div>
        </div>
        <menu-bar :ifTitleAndMenuShow="ifTitleAndMenuShow" 
                  :fontSizeList="fontSizeList"
                  :defaultFontSize="defaultFontSize"
                  @setFontSize="setFontSize" 
                  :themeList="themeList"
                  :defaultTheme="defaultTheme"
                  @setTheme="setTheme"
                  :bookAvailable="bookAvailable"
                  @onProgressChange="onProgressChange"
                  :navigation="navigation"
                  @jumpTo="jumpTo"
                  ref="menuBar"></menu-bar>
    </div>    
</template>
<script>
import TitleBar from '@/components/TitleBar'
import MenuBar from '@/components/MenuBar'
import Epub from 'epubjs'
const DOWNLOAD_URL = 'static/gamebird.epub' 
global.epub = Epub
export default {
    components:{
        TitleBar,
        MenuBar
    },
    data(){
        return {
            ifTitleAndMenuShow:false,
            fontSizeList:[
                {fontSize:12},
                {fontSize:14},
                {fontSize:16},
                {fontSize:18},
                {fontSize:20},
                {fontSize:22},
                {fontSize:24}
            ],
            defaultFontSize:16,
            themeList:[
                {
                    name:'default',
                    style:{
                        body:{
                            'color':'#000','background':'#fff'
                        }
                    }
                },
                {
                    name:'eye',
                    style:{
                        body:{
                            'color':'#000','background':'#ceeaba'
                        }
                    }
                },
                {
                    name:'night',
                    style:{
                        body:{
                            'color':'#fff','background':'#000'
                        }
                    }
                },
                {
                    name:'gold',
                    style:{
                        body:{
                            'color':'#000','background':'rgb(241,236,226)'
                        }
                    }
                }
            ],
            defaultTheme:0,
            bookAvailable:false, //电子书是否可用
            navigation:{}
        }
    },
    methods:{
        registerTheme(){
            this.themeList.forEach(theme=>{
                this.themes.register(theme.name,theme.style)
            })
        },
        setTheme(index){
            this.themes.select(this.themeList[index].name);
            this.defaultTheme = index;
        },
        setFontSize(fontSize){
            this.defaultFontSize = fontSize;
            if(this.themes){
                this.themes.fontSize(fontSize+"px");
            }
        },
        onProgressChange(progress){
            const percentage = progress/100;
            const location = percentage>0?this.locations.cfiFromPercentage(percentage):0;
            //定位到当前位置
            this.rendition.display(location);
        },
        //目录跳转
        jumpTo(href){
            this.rendition.display(href);
            this.hideTilteAndMenu();
            this.rendition.next().then(()=>{
                const currentLocation = this.rendition.currentLocation();
                this.progress = this.locations.percentageFromCfi(currentLocation.start.cfi);
                this.$refs.menuBar.progress = this.progress<1?this.progress.toFixed(2)*100:1;
            })
        },
        hideTilteAndMenu(){
            this.ifTitleAndMenuShow = false;
            this.$refs.menuBar.hideSetting();
            this.$refs.menuBar.hideContent();
        },
        toggleTitleAndMenu(){
            this.ifTitleAndMenuShow = !this.ifTitleAndMenuShow;
            if(!this.ifTitleAndMenuShow){
                this.$refs.menuBar.hideSetting();
            }
        },
        //电子书的解析和渲染
        showEpub (){
            //生成Book
            this.book = new Epub(DOWNLOAD_URL);
            //Rendition
            this.rendition = this.book.renderTo('read',{
                width:window.innerWidth,
                height:window.innerHeight
            });
            //Rendition.display
            this.rendition.display();
            //获取Theme
            this.themes = this.rendition.themes;
            this.setFontSize(this.defaultFontSize);
            //主题改变 
            //this.themes.register(name,styles)
            //this.themes.select(name)
            this.registerTheme();
            this.setTheme(this.defaultTheme);
            //ready 是当book对象初始化完后执行
            //locations 对象主要是定位
            //generate() 主要是生成分页
            this.book.ready.then(()=>{
                this.navigation = this.book.navigation;
                return this.book.locations.generate();
            }).then(()=>{
                this.locations = this.book.locations; 
                this.bookAvailable = true;
            })
        },
        prevPage(){
            if(this.rendition){
                this.rendition.prev();
            }
        },
        nextPage(){
            if(this.rendition){
                this.rendition.next();
            }
        }
    },
    mounted (){
        this.showEpub();
    }
}
</script>
<style lang="scss" scoped>
@import "assets/styles/global.scss";
.ebook{
    position: relative;
    .read-wrapper{
        .mask{
            position: absolute;
            top:0;
            left: 0;
            z-index: 100;
            display: flex;
            width: 100%;
            height: 100%;
            .left{
                flex: 0 0 px2rem(100);
            }
            .center{
                flex: 1;
            }
            .right{
                flex: 0 0 px2rem(100);
            }
        }
    }
    
}
</style>


