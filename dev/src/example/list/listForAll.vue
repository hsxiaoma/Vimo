<template>
    <Page>
        <Header>
            <Navbar>
                <Title>ListForAll</Title>
            </Navbar>
        </Header>
        <Content class="outer-content">
            <List>
                <!--header-->
                <ListHeader>
                    setting



                    <Button slot="item-right" clear>
                        <Icon name="cog"></Icon>
                    </Button>
                </ListHeader>
                <!--group-->
                <ItemGroup>
                    <Item>
                        <Icon slot="item-left" color="danger" name="plane"></Icon>
                        Airplane Mode



                        <Toggle slot="item-right" @ionChange="toggleChange"
                                :checked="toggleValue"></Toggle>
                    </Item>
                    <!--button属性会在右边增加向右箭头-->
                    <Item button>
                        <Icon slot="item-left" color="primary" name="wifi"></Icon>
                        Wi-Fi



                        <Note slot="item-right">The Interwebz</Note>
                    </Item>
                    <!--button属性会在右边增加向右箭头-->
                    <Item button>
                        <Icon slot="item-left" color="primary" name="bluetooth"></Icon>
                        Bluetooth



                        <Note slot="item-right">Off</Note>
                    </Item>
                </ItemGroup>
                <!--divider-->
                <ItemDivider color="primary">
                    Other Setting



                    <Button slot="item-right" outline color="light">Clear</Button>
                </ItemDivider>

                <!--button属性会在右边增加向右箭头-->
                <Item button>
                    <Icon slot="item-left" color="secondary" name="call"></Icon>
                    Cellular



                </Item>
                <!--button属性会在右边增加向右箭头-->
                <Item button>
                    <Icon slot="item-left" color="secondary" name="link"></Icon>
                    Personal Hotspot



                    <Note slot="item-right">Off</Note>
                </Item>
            </List>

            <List radio-group :value="silencePhone">
                <!--header-->
                <ListHeader>
                    <Icon slot="item-left" name="phone-portrait"></Icon>
                    Silence Phone



                </ListHeader>
                <!--button属性会在右边增加向右箭头-->
                <Item>
                    Always
                    <!--<ion-radio value="always" checked></ion-radio>-->
                </Item>
                <!--button属性会在右边增加向右箭头-->
                <Item>
                    Only while phone is locked



                </Item>
            </List>

            <List>
                <!--header-->
                <ListHeader>
                    <span>Apps Installed</span>
                </ListHeader>

                <!--button属性会在右边增加向右箭头-->
                <Item>
                    <Icon slot="item-left" color="primary" name="ionic"></Icon>
                    <span>Ionic View</span>
                    <Button slot="item-right" color="primary" outline>Uninstall</Button>
                </Item>
                <!--button属性会在右边增加向右箭头-->
                <Item>
                    <Icon slot="item-left" color="primary" name="brush"></Icon>
                    <span>Ionic Creator</span>
                    <Button slot="item-right" outline color="primary">Uninstall</Button>
                </Item>
                <!--button属性会在右边增加向右箭头-->
                <Item>
                    <Icon slot="item-left" color="dark" name="logo-octocat"></Icon>
                    <span>Hubstruck</span>
                    <Button slot="item-right" outline color="primary">Uninstall</Button>
                </Item>
                <!--button属性会在右边增加向右箭头-->
                <Item>
                    <Icon slot="item-left" color="danger" name="paw"></Icon>
                    <span>Barkpark</span>
                    <Button slot="item-right" outline color="primary">Uninstall</Button>
                </Item>
            </List>
        </Content>
    </Page>
</template>
<style scoped lang="scss">
    .main {

    }
</style>
<script type="text/ecmascript-6">
  import { List } from 'vimo/components/list'
  import { ListHeader, ItemGroup, Item, ItemSliding, ItemOptions, ItemDivider } from 'vimo/components/item'
  import { Note } from 'vimo/components/note'
  import { Avatar } from 'vimo/components/avatar'
  import { Label } from 'vimo/components/label'
  import { Toggle } from 'vimo/components/toggle'
  export default{
    data(){
      return {
        toggleValue: true,
        silencePhone: 'always',
      }
    },
    watch: {
      toggleValue: function () {
        console.debug('outer toggleValue change:' + this.toggleValue)
      }
    },
    computed: {},
    methods: {
      toggleChange: function (val) {
        console.debug('Outer Toggle Change Value - airPlane:' + val)
      }
    },
    created: function () {
    },
    mounted: function () {
    },
    activated: function () {
    },
    components: {List, ListHeader, ItemGroup, Item, ItemSliding, ItemOptions, ItemDivider, Note, Avatar, Label, Toggle}
  }
</script>
