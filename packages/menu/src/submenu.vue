<script>
  import ElCollapseTransition from 'element-ui/src/transitions/collapse-transition';
  import menuMixin from './menu-mixin';
  import Emitter from 'element-ui/src/mixins/emitter';
  import Popper from 'element-ui/src/utils/vue-popper';

  const poperMixins = {
    props: {
      transformOrigin: {
        type: [Boolean, String],
        default: false
      },
      offset: Popper.props.offset,
      boundariesPadding: Popper.props.boundariesPadding,
      popperOptions: Popper.props.popperOptions
    },
    data: Popper.data,
    methods: Popper.methods,
    beforeDestroy: Popper.beforeDestroy,
    deactivated: Popper.deactivated
  };

  export default {
    name: 'ElSubmenu',

    componentName: 'ElSubmenu',

    mixins: [menuMixin, Emitter, poperMixins],

    components: { ElCollapseTransition },

    props: {
      index: {
        type: String,
        required: true
      },
      showTimeout: {
        type: Number,
        default: 300
      },
      hideTimeout: {
        type: Number,
        default: 300
      },
      arrows: {
        type: String,
        default: "right"
      },
      popperClass: String
    },

    data() {
      return {
        popperJS: null,
        timeout: null,
        items: {},
        submenus: {}
      };
    },
    watch: {
      opened(val) {
        if (this.isMenuPopup) {
          this.$nextTick(_ => {
            this.updatePopper();
          });
        }
      }
    },
    computed: {
      // popper option
      appendToBody() {
        return this.rootMenu === this.$parent;
      },
      menuTransitionName() {
        return this.rootMenu.collapse ? 'el-zoom-in-left' : 'el-zoom-in-top';
      },
      opened() {
        return this.rootMenu.openedMenus.indexOf(this.index) > -1;
      },
      active() {
        let isActive = false;
        const submenus = this.submenus;
        const items = this.items;

        Object.keys(items).forEach(index => {
          if (items[index].active) {
            isActive = true;
          }
        });

        Object.keys(submenus).forEach(index => {
          if (submenus[index].active) {
            isActive = true;
          }
        });

        return isActive;
      },
      hoverBackground() {
        return this.rootMenu.hoverBackground;
      },
      backgroundColor() {
        return this.rootMenu.backgroundColor || '';
      },
      activeTextColor() {
        return this.rootMenu.activeTextColor || '';
      },
      textColor() {
        return this.rootMenu.textColor || '';
      },
      mode() {
        return this.rootMenu.mode;
      },
      isMenuPopup() {
        return this.rootMenu.isMenuPopup;
      },
      titleStyle() {
        if (this.mode !== 'horizontal') {
          return {
            color: this.textColor
          };
        }
        return {
          borderBottomColor: this.active
            ? (this.rootMenu.activeTextColor ? this.activeTextColor : '')
            : 'transparent',
          color: this.active
            ? this.activeTextColor
            : this.textColor
        };
      }
    },
    methods: {
      handleCollapseToggle(value) {
        if (value) {
          this.initPopper();
        } else {
          this.doDestroy();
        }
      },
      addItem(item) {
        this.$set(this.items, item.index, item);
      },
      removeItem(item) {
        delete this.items[item.index];
      },
      addSubmenu(item) {
        this.$set(this.submenus, item.index, item);
      },
      removeSubmenu(item) {
        delete this.submenus[item.index];
      },
      handleClick() {
        const {rootMenu} = this;
        if (
          (rootMenu.menuTrigger === 'hover' && rootMenu.mode === 'horizontal') ||
          (rootMenu.collapse && rootMenu.mode === 'vertical')
        ) {
          return;
        }
        this.dispatch('ElMenu', 'submenu-click', this);
      },
      handleMouseenter() {
        const {rootMenu} = this;
        if (
          (rootMenu.menuTrigger === 'click' && rootMenu.mode === 'horizontal') ||
          (!rootMenu.collapse && rootMenu.mode === 'vertical')
        ) {
          return;
        }
        clearTimeout(this.timeout);
        this.timeout = setTimeout(() => {
          this.rootMenu.openMenu(this.index, this.indexPath);
        }, this.showTimeout);
      },
      handleMouseleave() {
        const {rootMenu} = this;
        if (
          (rootMenu.menuTrigger === 'click' && rootMenu.mode === 'horizontal') ||
          (!rootMenu.collapse && rootMenu.mode === 'vertical')
        ) {
          return;
        }
        clearTimeout(this.timeout);
        this.timeout = setTimeout(() => {
          this.rootMenu.closeMenu(this.index);
        }, this.hideTimeout);
      },
      handleTitleMouseenter() {
        if (this.mode === 'horizontal' && !this.rootMenu.backgroundColor) return;
        const title = this.$refs['submenu-title'];
        title && (title.style.backgroundColor = this.rootMenu.hoverBackground);
      },
      handleTitleMouseleave() {
        if (this.mode === 'horizontal' && !this.rootMenu.backgroundColor) return;
        const title = this.$refs['submenu-title'];
        title && (title.style.backgroundColor = this.rootMenu.backgroundColor || '');
      },
      updatePlacement() {
        this.currentPlacement = this.mode === 'horizontal' ? 'bottom-start' : 'right-start';
      },
      initPopper() {
        this.referenceElm = this.$el;
        this.popperElm = this.$refs.menu;
        this.updatePlacement();
      }
    },
    created() {
      this.parentMenu.addSubmenu(this);
      this.rootMenu.addSubmenu(this);
      this.$on('toggle-collapse', this.handleCollapseToggle);
    },
    mounted() {
      this.initPopper();
    },
    beforeDestroy() {
      this.parentMenu.removeSubmenu(this);
      this.rootMenu.removeSubmenu(this);
    },
    render(h) {
      const {
        active,
        opened,
        paddingStyle,
        titleStyle,
        backgroundColor,
        $slots,
        rootMenu,
        currentPlacement,
        menuTransitionName,
        mode,
        popperClass
      } = this;

      const popupMenu = (
        <transition name={menuTransitionName}>
          <div
            ref="menu"
            v-show={opened}
            class={[`el-menu--${mode}`, popperClass]}
            on-mouseenter={this.handleMouseenter}
            on-mouseleave={this.handleMouseleave}
            on-focus={this.handleMouseenter}>
            <ul
              role="menu"
              class={['el-menu el-menu--popup', `el-menu--popup-${currentPlacement}`]}
              style={{ backgroundColor: rootMenu.backgroundColor || '' }}>
              {$slots.default}
            </ul>
          </div>
        </transition>
      );

      const inlineMenu = (
        <el-collapse-transition>
          <ul
            role="menu"
            class="el-menu el-menu--inline"
            v-show={opened}
            style={{ backgroundColor: rootMenu.backgroundColor || '' }}>
            {$slots.default}
          </ul>
        </el-collapse-transition>
      );

      return (
        <li
          class={{
            'el-submenu': true,
            'is-active': active,
            'is-opened': opened
          }}
          role="menuitem"
          aria-haspopup="true"
          aria-expanded={opened}
          on-mouseenter={this.handleMouseenter}
          on-mouseleave={this.handleMouseleave}
          on-focus={this.handleMouseenter}
        >
          <i v-if='arrows=="left"' class={{
            'el-submenu__icon-arrow': true,
            'el-icon-arrow-down': rootMenu.mode === 'horizontal' || rootMenu.mode === 'vertical' && !rootMenu.collapse,
            'el-icon-arrow-right': rootMenu.mode === 'vertical' && rootMenu.collapse
          }}>
          </i>
          <div
            class="el-submenu__title"
            ref="submenu-title"
            on-click={this.handleClick}
            on-mouseenter={this.handleTitleMouseenter}
            on-mouseleave={this.handleTitleMouseleave}
            style={[paddingStyle, titleStyle, { backgroundColor }]}
          >
            {$slots.title}
            
            <i v-if='arrows=="right"' class={{
              'el-submenu__icon-arrow': true,
              'el-icon-arrow-down': rootMenu.mode === 'horizontal' || rootMenu.mode === 'vertical' && !rootMenu.collapse,
              'el-icon-arrow-right': rootMenu.mode === 'vertical' && rootMenu.collapse
            }}>
            </i>
          </div>
          {this.isMenuPopup ? popupMenu : inlineMenu}
        </li>
      );
    }
  };
</script>
