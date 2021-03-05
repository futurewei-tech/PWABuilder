<template>
  <main id="main">
    <HubHeader></HubHeader>

    <div
      v-if="openAndroid || openWindows || openTeams || showBackground"
      class="has-acrylic-40 is-dark"
      id="modalBackground"
    ></div>

    <!-- Toast notification for package download errors -->
    <div
      class="packageError"
      role="alert"
      aria-live="assertive"
      aria-atomic="true"
      v-if="packageErrorMessage"
    >
      <div class="errorHeader">
        <strong class="errorTitle">
          <i class="fa fa-exclamation-circle text-danger"></i>
          Error creating package
        </strong>
        <button
          type="button"
          class="closeBtn"
          aria-label="Close"
          @click="packageErrorMessage = null"
        >
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="errorBody">
        <pre>{{packageErrorMessage}}</pre>
      </div>
      <div class="errorFooter">
        <a target="_blank" rel="noopener" :href="reportPackageErrorUrl">
          <i class="fa fa-bug"></i>
          Report a problem
        </a>
      </div>
    </div>    

    <!-- Toast notification for AppGallery publish success -->
    <div class="packageError" role="alert" aria-live="assertive" aria-atomic="true" v-if="appGalleryPublishSuccessMessage">
      <div class="errorHeader">
        <strong class="errorTitle">
          <i class="fa fa-check-circle text-success"></i>
          AppGallery Publish Success!
        </strong>
        <button type="button" class="closeBtn" aria-label="Close" @click="appGalleryPublishSuccessMessage = null">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="errorBody">
        <pre>{{appGalleryPublishSuccessMessage}}</pre>
      </div>
      <div class="errorFooter">

      </div>
    </div>

    <Modal
      :title="$t('publish.package_name')"
      :button_name="$t('modal.done')"
      ref="androidPWAModal"
      @modalSubmit="androidOptionsModalSubmitted"
      @cancel="androidOptionsModalCancelled"
      v-on:modalOpened="modalOpened()"
      v-on:modalClosed="androidModalClosed()"
      v-if="androidForm"
    >
      <div id="topLabelBox" slot="extraP">
        <label id="topLabel">{{ $t("publish.package_name_detail") }}</label>
        <ul class="l-generator-error" v-if="androidPWAErrors.length">
          <li v-for="error in androidPWAErrors" v-bind:key="error">
            <i class="fas fa-exclamation-circle"></i>
            &nbsp;
            <span>{{ $t(error) }}</span>
          </li>
        </ul>
      </div>

      <section class="androidModalBody androidOptionsModalBody">
        <form style="width: 100%">
          <div class="row">
            <div class="col-lg-6 col-md-12">
              <div class="form-group">
                <label for="packageIdInput">
                  {{ $t("publish.label_package_name") }}
                  <i
                    class="fas fa-info-circle"
                    title="The unique identifier of your app. It should contain only letters, numbers, and periods. Example: com.companyname.appname"
                    aria-label="The unique identifier of your app. It should contain only letters, numbers, and periods. Example: com.companyname.appname"
                    role="definition"
                  ></i>
                </label>
                <input
                  id="packageIdInput"
                  class="form-control"
                  :placeholder="$t('publish.placeholder_package_name')"
                  type="text"
                  required
                  v-model="androidForm.packageId"
                />
              </div>

              <div class="row">
                <div class="col-lg-6 col-md-12">
                  <div class="form-group">
                    <label for="appNameInput">App name</label>
                    <input
                      type="text"
                      class="form-control"
                      id="appNameInput"
                      placeholder="My Awesome PWA"
                      required
                      v-model="androidForm.name"
                    />
                  </div>
                </div>

                <div class="col-lg-6 col-md-12">
                  <div class="form-group">
                    <label for="appLauncherNameInput">
                      Launcher name
                      <i
                        class="fas fa-info-circle"
                        title="The app name used on the Android launch screen. Typically, this is the short name of the app."
                        aria-label="The app name used on the Android launch screen. Typically, this is the short name of the app."
                        role="definition"
                      ></i>
                    </label>
                    <input
                      type="text"
                      class="form-control"
                      id="appLauncherNameInput"
                      placeholder="Awesome PWA"
                      required
                      v-model="androidForm.launcherName"
                    />
                  </div>
                </div>
              </div>

              <div class="row">
                <div class="col-lg-6 col-md-12">
                  <div class="form-group">
                    <label for="appVersionInput">
                      App version
                      <i
                        class="fas fa-info-circle"
                        title="The version of your app displayed to users. This is a string, typically in the form of '1.0.0.0'. Maps to android:versionName."
                        aria-label
                        role="definition"
                      ></i>
                    </label>
                    <input
                      type="text"
                      class="form-control"
                      id="appVersionInput"
                      placeholder="1.0.0.0"
                      required
                      v-model="androidForm.appVersion"
                    />
                  </div>
                </div>

                <div class="col-lg-6 col-md-12">
                  <div class="form-group">
                    <label for="appVersionCodeInput">
                      <a
                        href="https://developer.android.com/studio/publish/versioning#appversioning"
                        target="_blank"
                        rel="noopener"
                      >App version code</a>
                      <i
                        class="fas fa-info-circle"
                        title="A positive integer used as an internal version number. This is not shown to users. Android uses this value to protect against downgrades. Maps to android:versionCode."
                        aria-label="A positive integer used as an internal version number. This is not shown to users. Android uses this value to protect against downgrades. Maps to android:versionCode."
                        role="definition"
                        style="margin-left: 5px;"
                      ></i>
                    </label>
                    <input
                      type="number"
                      min="1"
                      max="2100000000"
                      class="form-control"
                      id="appVersionCodeInput"
                      placeholder="1"
                      required
                      v-model="androidForm.appVersionCode"
                    />
                  </div>
                </div>
              </div>

              <div class="form-group">
                <label for="hostInput">Host</label>
                <input
                  type="url"
                  class="form-control"
                  id="hostInput"
                  placeholder="https://mysite.com"
                  required
                  v-model="androidForm.host"
                />
              </div>

              <div class="form-group">
                <label for="startUrlInput">
                  Start URL
                  <i
                    class="fas fa-info-circle"
                    title="The start path for the TWA. Must be relative to the Host URL. You can specify '/' if you don't have a start URL different from Host."
                    aria-label="The start path for the TWA. Must be relative to the Host URL."
                    role="definition"
                  ></i>
                </label>
                <input
                  type="url"
                  class="form-control"
                  id="startUrlInput"
                  placeholder="/index.html"
                  required
                  v-model="androidForm.startUrl"
                />
              </div>

              <div class="form-group">
                <label for="manifestUrlInput">Manifest URL</label>
                <input
                  type="url"
                  class="form-control"
                  id="manifestUrlInput"
                  placeholder="https://mysite.com/manifest.json"
                  required
                  v-model="androidForm.webManifestUrl"
                />
              </div>
              <div class="row">
                <div class="col-lg-6 col-md-12">
                  <div class="form-group">
                    <label for="themeColorInput">
                      Status bar color
                      <i
                        class="fas fa-info-circle"
                        title="Also known as the theme color, this is the color of the Android status bar in your app. Note: the status bar will be hidden if Display Mode is set to fullscreen."
                        aria-label="Also known as the theme color, this is the color of the Android status bar in your app. Note: the status bar will be hidden if Display Mode is set to fullscreen."
                        role="definition"
                      ></i>
                    </label>
                    <input
                      type="color"
                      class="form-control"
                      id="themeColorInput"
                      v-model="androidForm.themeColor"
                    />
                  </div>
                </div>

                <div class="col-lg-6 col-md-12">
                  <div class="form-group">
                    <label for="bgColorInput">
                      Splash color
                      <i
                        class="fas fa-info-circle"
                        title="Also known as background color, this is the color of the splash screen for your app."
                        aria-label="Also known as background color, this is the color of the splash screen for your app."
                        role="definition"
                      ></i>
                    </label>
                    <input
                      type="color"
                      class="form-control"
                      id="bgColorInput"
                      v-model="androidForm.backgroundColor"
                    />
                  </div>
                </div>
              </div>

              <!-- second row of colors -->
              <div class="row">

                <!-- Nav bar color -->
                <div class="col-lg-6 col-md-12">
                  <div class="form-group">
                    <label for="navigationColorInput">
                      Nav color
                      <i
                        class="fas fa-info-circle"
                        title="The color of the Android navigation bar in your app. Note: the navigation bar will be hidden if Display Mode is set to fullscreen."
                        aria-label="The color of the Android navigation bar in your app. Note: the navigation bar will be hidden if Display Mode is set to fullscreen."
                        role="definition"
                      ></i>
                    </label>
                    <input
                      type="color"
                      class="form-control"
                      id="navigationColorInput"
                      v-model="androidForm.navigationColor"
                    />
                  </div>
                </div>

                <!-- Nav bar dark color -->
                <div class="col-lg-6 col-md-12">
                  <div class="form-group">
                    <label for="navigationColorDarkInput">
                      Nav dark color
                      <i
                        class="fas fa-info-circle"
                        title="The color of the Android navigation bar in your app when Android is in dark mode."
                        aria-label="The color of the Android navigation bar in your app when Android is in dark mode."
                        role="definition"
                      ></i>
                    </label>
                    <input
                      type="color"
                      class="form-control"
                      id="navigationColorDarkInput"
                      v-model="androidForm.navigationColorDark"
                    />
                  </div>
                </div>
              </div>

              <!-- third row of colors -->
              <div class="row">

                <!-- Nav bar divider color -->
                <div class="col-lg-6 col-md-12">
                  <div class="form-group">
                    <label for="navigationDividerColorInput">
                      Nav divider color
                      <i
                        class="fas fa-info-circle"
                        title="The color of the Android navigation bar divider in your app."
                        aria-label="The color of the Android navigation bar divider in your app."
                        role="definition"
                      ></i>
                    </label>
                    <input
                      type="color"
                      class="form-control"
                      id="navigationDividerColorInput"
                      v-model="androidForm.navigationDividerColor"
                    />
                  </div>
                </div>

                <!-- Nav bar divider dark color -->
                <div class="col-lg-6 col-md-12">
                  <div class="form-group">
                    <label for="navigationDividerColorDarkInput">
                      Nav divider dark color
                      <i
                        class="fas fa-info-circle"
                        title="The color of the Android navigation navigation bar divider in your app when Android is in dark mode."
                        aria-label="The color of the Android navigation bar divider in your app when Android is in dark mode."
                        role="definition"
                      ></i>
                    </label>
                    <input
                      type="color"
                      class="form-control"
                      id="navigationDividerColorDarkInput"
                      v-model="androidForm.navigationDividerColorDark"
                    />
                  </div>
                </div>
              </div>

            </div>

            <!-- right half of the options dialog -->
            <div class="col-lg-6 col-md-12">
              <div class="form-group">
                <label for="iconUrlInput">Icon URL</label>
                <input
                  type="url"
                  class="form-control"
                  id="iconUrlInput"
                  placeholder="https://myawesomepwa.com/512x512.png"
                  v-model="androidForm.iconUrl"
                />
              </div>

              <div class="form-group">
                <label for="maskIconUrlInput">
                  <a
                    href="https://web.dev/maskable-icon"
                    title="Read more about maskable icons"
                    target="_blank"
                    rel="noopener"
                    aria-label="Read more about maskable icons"
                  >Maskable icon</a> URL
                  <i
                    class="fas fa-info-circle"
                    title="Optional. The URL to an icon with a minimum safe zone of trimmable padding, enabling rounded icons on certain Android platforms."
                    aria-label="Optional. The URL to an icon with a minimum safe zone of trimmable padding, enabling rounded icons on certain Android platforms."
                    role="definition"
                  ></i>
                </label>
                <input
                  type="url"
                  class="form-control"
                  id="maskIconUrlInput"
                  placeholder="https://myawesomepwa.com/512x512-maskable.png"
                  v-model="androidForm.maskableIconUrl"
                />
              </div>

              <div class="form-group">
                <label for="monochromeIconUrlInput">
                  <a
                    href="https://w3c.github.io/manifest/#monochrome-icons-and-solid-fills"
                    target="_blank"
                    rel="noopener"
                  >Monochrome icon</a> URL
                  <i
                    class="fas fa-info-circle"
                    title="Optional. The URL to an icon containing only white and black colors, enabling Android to fill the icon with user-specified color or gradient depending on theme, color mode, or contrast settings."
                    aria-label="Optional. The URL to an icon containing only white and black colors, enabling Android to fill the icon with user-specified color or gradient depending on theme, color mode, or contrast settings."
                    role="definition"
                  ></i>
                </label>
                <input
                  type="url"
                  class="form-control"
                  id="monochromeIconUrlInput"
                  placeholder="https://myawesomepwa.com/512x512-monochrome.png"
                  v-model="androidForm.monochromeIconUrl"
                />
              </div>

              <div class="form-group">
                <label for="splashFadeoutInput">Splash screen fade out duration (ms)</label>
                <input
                  type="number"
                  class="form-control"
                  id="splashFadeoutInput"
                  placeholder="300"
                  v-model="androidForm.splashScreenFadeOutDuration"
                />
              </div>

              <div class="form-group">
                <label>Fallback behavior</label>
                <div class="form-check">
                  <input
                    class="form-check-input"
                    type="radio"
                    name="fallbackType"
                    id="fallbackCustomTabsInput"
                    value="customtabs"
                    v-model="androidForm.fallbackType"
                  />
                  <label class="form-check-label" for="fallbackCustomTabsInput">
                    Custom Tabs
                    <i
                      class="fas fa-info-circle"
                      title="Use Chrome Custom Tabs as a fallback for your PWA when the full trusted web activity (TWA) experience is unavailable."
                      aria-label="When trusted web activity (TWA) is unavailable, use Chrome Custom Tabs as a fallback for your PWA."
                      role="definition"
                    ></i>
                  </label>
                </div>
                <div class="form-check">
                  <input
                    class="form-check-input"
                    type="radio"
                    name="fallbackType"
                    id="fallbackWebViewInput"
                    value="webview"
                    v-model="androidForm.fallbackType"
                  />
                  <label class="form-check-label" for="fallbackWebViewInput">
                    Web View
                    <i
                      class="fas fa-info-circle"
                      title="Use a web view as the fallback for your PWA when the full trusted web activity (TWA) experience is unavailable."
                      aria-label="When trusted web activity (TWA) is unavailable, use a web view as the fallback for your PWA."
                      role="definition"
                    ></i>
                  </label>
                </div>
              </div>

              <div class="form-group">
                <label>Display mode</label>
                <div class="form-check">
                  <input
                    class="form-check-input"
                    type="radio"
                    name="displayMode"
                    id="standaloneDisplayModeInput"
                    value="standalone"
                    v-model="androidForm.display"
                  />
                  <label class="form-check-label" for="standaloneDisplayModeInput">
                    Standalone
                    <i
                      class="fas fa-info-circle"
                      title="Your PWA will use the whole screen but keep the Android status bar and navigation bar."
                      aria-label="Your PWA will use the whole screen but keep the Android status bar and navigation bar."
                      role="definition"
                    ></i>
                  </label>
                </div>
                <div class="form-check">
                  <input
                    class="form-check-input"
                    type="radio"
                    name="displayMode"
                    id="fullscreenDisplayModeInput"
                    value="fullscreen"
                    v-model="androidForm.display"
                  />
                  <label class="form-check-label" for="fullscreenDisplayModeInput">
                    Fullscreen
                    <i
                      class="fas fa-info-circle"
                      title="Your PWA will use the whole screen and remove the Android status bar and navigation bar. Suitable for immersive experiences such as games or media apps."
                      aria-label="Your PWA will use the whole screen and remove the Android status bar and navigation bar. Suitable for immersive experiences such as games or media apps."
                      role="definition"
                    ></i>
                  </label>
                </div>
              </div>

              <div class="form-group">
                <label>Notification delegation</label>
                <div class="form-check">
                  <input
                    class="form-check-input"
                    type="checkbox"
                    id="enableNotificationsInput"
                    v-model="androidForm.enableNotifications"
                  />
                  <label class="form-check-label" for="enableNotificationsInput">
                    Enable
                    <i
                      class="fas fa-info-circle"
                      title="Whether to enable Push Notification Delegation. If enabled, your PWA can send push notifications without browser permission prompts."
                      aria-label="Whether to enable Push Notification Delegation. If enabled, your PWA can send push notifications without browser permission prompts."
                      role="definition"
                    ></i>
                  </label>
                </div>
              </div>

              <div class="form-group">
                <label>Location delegation</label>
                <div class="form-check">
                  <input
                    class="form-check-input"
                    type="checkbox"
                    id="enableLocationInput"
                    v-model="androidForm.features.locationDelegation.enabled"
                  />
                  <label class="form-check-label" for="enableLocationInput">
                    Enable
                    <i
                      class="fas fa-info-circle"
                      title="Whether to enable Location Delegation. If enabled, your PWA can acess navigator.geolocation without browser permission prompts."
                      aria-label="Whether to enable Location Delegation. If enabled, your PWA can acess navigator.geolocation without browser permission prompts."
                      role="definition"
                    ></i>
                  </label>
                </div>
              </div>

              <div class="form-group">
                <label>Google Play billing</label>
                <div class="form-check">
                  <input
                    class="form-check-input"
                    type="checkbox"
                    id="enablePlayBillingInput"
                    v-model="androidForm.features.playBilling.enabled"
                  />
                  <label class="form-check-label" for="enablePlayBillingInput">
                    Enable
                    <i
                      class="fas fa-info-circle"
                      title="Whether to enable in-app purchases through Google Play Billing and the Digital Goods API."
                      aria-label="Whether to enable in-app purchases through Google Play Billing and the Digital Goods API."
                      role="definition"
                    ></i>
                  </label>
                </div>
              </div>

              <div class="form-group">
                <label>Settings shortcut</label>
                <div class="form-check">
                  <input
                    class="form-check-input"
                    type="checkbox"
                    id="enableSettingsShortcutInput"
                    v-model="androidForm.enableSiteSettingsShortcut"
                  />
                  <label class="form-check-label" for="enableSettingsShortcutInput">
                    Enable
                    <i
                      class="fas fa-info-circle"
                      title="If enabled, users can long-press on your app tile and a Settings menu item will appear, letting users manage space for your app."
                      aria-label="If enabled, users can long-press on your app tile and a Settings menu item will appear, letting users manage space for your app."
                      role="definition"
                    ></i>
                  </label>
                </div>
              </div>

              <div class="form-group">
                <label>ChromeOS only</label>
                <div class="form-check">
                  <input
                    class="form-check-input"
                    type="checkbox"
                    id="chromeOSOnlyInput"
                    v-model="androidForm.isChromeOSOnly"
                  />
                  <label class="form-check-label" for="chromeOSOnlyInput">
                    Enable
                    <i
                      class="fas fa-info-circle"
                      title="If enabled, your Android package will only run on ChromeOS devices"
                      aria-label="If enabled, your Android package will only run on ChromeOS devices"
                      role="definition"
                    ></i>
                  </label>
                </div>
              </div>

              <div class="form-group">
                <label>Include source code</label>
                <div class="form-check">
                  <input
                    class="form-check-input"
                    type="checkbox"
                    id="includeSourceCodeInput"
                    v-model="androidForm.includeSourceCode"
                  />
                  <label class="form-check-label" for="includeSourceCodeInput">
                    Enable
                    <i
                      class="fas fa-info-circle"
                      title="If enabled, your download will include the source code for your Android app."
                      aria-label="If enabled, your download will include the source code for your Android app."
                      role="definition"
                    ></i>
                  </label>
                </div>
              </div>

              <div class="form-group">
                <label>Signing key</label>
                <div class="form-check">
                  <input
                    class="form-check-input"
                    type="radio"
                    name="signingInput"
                    id="generateSigningKeyInput"
                    value="new"
                    v-model="androidForm.signingMode"
                    @change="androidSigningModeChanged"
                  />
                  <label class="form-check-label" for="generateSigningKeyInput">
                    Create new
                    <i
                      class="fas fa-info-circle"
                      title="PWABuilder will generate a new signing key for you and sign your APK with it. Your download will contain the new signing key and passwords."
                      aria-label="PWABuilder will generate a new signing key for you and sign your APK with it. Your download will contain the new signing key and passwords."
                      role="definition"
                    ></i>
                  </label>
                </div>
                <div class="form-check">
                  <input
                    class="form-check-input"
                    type="radio"
                    name="signingInput"
                    id="unsignedInput"
                    value="none"
                    v-model="androidForm.signingMode"
                    @change="androidSigningModeChanged"
                  />
                  <label class="form-check-label" for="unsignedInput">
                    None
                    <i
                      class="fas fa-info-circle"
                      title="PWABuilder will generate an unsigned APK. Google Play Store will sign your package. This is Google's recommended approach."
                      aria-label="PWABuilder will generate an unsigned APK. Google Play Store will sign your package. This is Google's recommended approach."
                      role="definition"
                    ></i>
                  </label>
                </div>
                <div class="form-check">
                  <input
                    class="form-check-input"
                    type="radio"
                    name="signingInput"
                    id="useMySigningInput"
                    value="mine"
                    v-model="androidForm.signingMode"
                    @change="androidSigningModeChanged"
                  />
                  <label class="form-check-label" for="useMySigningInput">
                    Use mine
                    <i
                      class="fas fa-info-circle"
                      title="Upload your existing signing key. Use this option if you already have a signing key and you want to publish a new version of an existing app in Google Play."
                      aria-label="Upload your existing signing key. Use this option if you already have a signing key and you want to publish a new version of an existing app in Google Play."
                      role="definition"
                    ></i>
                  </label>
                </div>
              </div>

              <div
                v-if="androidForm.signingMode === 'mine' || androidForm.signingMode === 'new'"
                style="margin-left: 15px;"
              >
                <div class="form-group" v-if="androidForm.signingMode === 'mine'">
                  <label for="signingKeyInput">Key file</label>
                  <input
                    type="file"
                    class="form-control"
                    id="signingKeyInput"
                    @change="androidSigningKeyUploaded"
                    accept=".keystore"
                    required
                    style="border: none;"
                  />
                </div>

                <div class="form-group">
                  <label for="signingKeyAliasInput">Key alias</label>
                  <input
                    type="text"
                    class="form-control"
                    id="signingKeyAliasInput"
                    placeholder="my-key-alias"
                    required
                    v-model="androidForm.signing.alias"
                  />
                </div>

                <div class="form-group" v-if="androidForm.signingMode === 'new'">
                  <label for="signingKeyFullNameInput">Key full name</label>
                  <input
                    type="text"
                    class="form-control"
                    id="signingKeyFullNameInput"
                    required
                    placeholder="John Doe"
                    v-model="androidForm.signing.fullName"
                  />
                </div>

                <div class="form-group" v-if="androidForm.signingMode === 'new'">
                  <label for="signingKeyOrgInput">Key organization</label>
                  <input
                    type="text"
                    class="form-control"
                    id="signingKeyOrgInput"
                    required
                    placeholder="My Company"
                    v-model="androidForm.signing.organization"
                  />
                </div>

                <div class="form-group" v-if="androidForm.signingMode === 'new'">
                  <label for="signingKeyOrgUnitInput">Key organizational unit</label>
                  <input
                    type="text"
                    class="form-control"
                    id="signingKeyOrgUnitInput"
                    required
                    placeholder="Engineering Department"
                    v-model="androidForm.signing.organizationalUnit"
                  />
                </div>

                <div class="form-group" v-if="androidForm.signingMode === 'new'">
                  <label for="signingKeyCountryCodeInput">
                    Key country code
                    <i
                      class="fas fa-info-circle"
                      title="The 2 letter country code to list on the signing key"
                      aria-label="The 2 letter country code to list on the signing key"
                      role="definition"
                    ></i>
                  </label>
                  <input
                    type="text"
                    class="form-control"
                    id="signingKeyCountryCodeInput"
                    required
                    placeholder="US"
                    v-model="androidForm.signing.countryCode"
                  />
                </div>

                <div class="form-group">
                  <label for="signingKeyPasswordInput">
                    Key password
                    <i
                        class="fas fa-info-circle"
                        title="The password for the signing key. Type a new password or leave empty to use a generated password."
                        aria-label="The password for the signing key. Type a new password or leave empty to use a generated password."
                        role="definition"
                      ></i>
                  </label>
                  <input
                    type="password"
                    class="form-control"
                    id="signingKeyPasswordInput"
                    v-model="androidForm.signing.keyPassword"
                    :placeholder="androidPasswordPlaceholder"
                  />
                </div>

                <div class="form-group">
                  <label for="signingKeyStorePasswordInput">
                    Key store password
                    <i
                        class="fas fa-info-circle"
                        title="The password for the key store. Type a new password or leave empty to use a generated password."
                        aria-label="The password for the key store. Type a new password or leave empty to use a generated password."
                        role="definition"
                      ></i>
                  </label>
                  <input
                    type="password"
                    class="form-control"
                    id="signingKeyStorePasswordInput"
                    v-model="androidForm.signing.storePassword"
                    :placeholder="androidPasswordPlaceholder"
                  />
                </div>
              </div>
            </div>
          </div>
        </form>
      </section>
    </Modal>

    <!-- New Edge Modal -->

    <Modal
      title="Windows Package Options"
      :button_name="$t('modal.done')"
      ref="windowsPWAModal"
      :showSubmitButton="true"
      @modalSubmit="windowsOptionsModalSubmitted"
      @cancel="windowsOptionsModalCancelled"
      v-on:modalOpened="modalOpened()"
      v-on:modalClosed="windowsModalClosed()"
      v-if="windowsForm"
    >
      <div id="topLabelBox" slot="extraP">
        <label id="topLabel">Customize your Windows package below</label>
        <ul class="l-generator-error" v-if="windowsOptionsErrors.length">
          <li v-for="error in windowsOptionsErrors" v-bind:key="error">
            <i class="fas fa-exclamation-circle"></i>
            &nbsp;
            <span>{{ $t(error) }}</span>
          </li>
        </ul>
      </div>

      <section class="androidModalBody androidOptionsModalBody">
        <form style="width: 100%">
          <div class="row">
            <div class="col-lg-6 col-md-12">
              <div class="form-group">
                <label for="windowsPackageIdInput">
                  <a target="_blank" href="https://github.com/pwa-builder/pwabuilder-windows-chromium-docs/blob/master/find-publisher.md">
                    {{ $t("publish.label_package_name") }}
                    <i
                      class="fas fa-info-circle"
                      title="The Microsoft Store's unique identifier for your app. You can find this value in Windows Partner Center. Click to learn more."
                      aria-label="The Microsoft Store's unique identifier for your app. You can find this value in Windows Partner Center. Click to learn more."
                      role="definition"
                    ></i>
                  </a>
                </label>
                <input
                  id="windowsPackageIdInput"
                  class="form-control"
                  :placeholder="$t('publish.placeholder_package_name')"
                  type="text"
                  required
                  v-model="windowsForm.packageId"
                />
              </div>

              <div class="row">
                <div class="col-lg-6 col-md-12">
                  <div class="form-group">
                    <label for="windowsAppNameInput">App name</label>
                    <input
                      type="text"
                      class="form-control"
                      id="windowsAppNameInput"
                      placeholder="My Awesome PWA"
                      required
                      v-model="windowsForm.name"
                    />
                  </div>
                </div>
              </div>

              <div class="row">
                <div class="col-lg-6 col-md-12">
                  <div class="form-group">
                    <label for="windowsAppVersionInput">
                      <a target="_blank" href="https://github.com/pwa-builder/pwabuilder-windows-chromium-docs/blob/master/classic-package.md">
                        App version
                        <i
                          class="fas fa-info-circle"
                          title="Your app version in the form of '1.0.0'. This must be greater than classic package version. Click to learn more."
                          aria-label="Your app version in the form of '1.0.0'. This must be greater than classic package version. Click to learn more."
                          role="definition"
                        ></i>
                      </a>
                    </label>
                    <input
                      type="text"
                      class="form-control"
                      id="windowsAppVersionInput"
                      placeholder="1.0.1"
                      required
                      v-model="windowsForm.version"
                    />
                  </div>
                </div>

              </div>

              <div class="row" v-if="windowsFormConfiguration === 'anaheim'">
                <div class="col-lg-6 col-md-12">
                  <div class="form-group">
                    <label for="windowsClassicAppVersionInput">
                      <a target="_blank" href="https://github.com/pwa-builder/pwabuilder-windows-chromium-docs/blob/master/classic-package.md">
                        Classic package version
                        <i
                          class="fas fa-info-circle"
                          title="The version of your app that runs on older versions of Windows. Must be in the form of '1.0.0'. Must be less than app version. Click to learn more."
                          aria-label="The version of your app that runs on older versions of Windows. Must be in the form of '1.0.0'. Must be less than app version. Click to learn more."
                          role="definition"
                        ></i>
                      </a>
                    </label>
                    <input
                      type="text"
                      class="form-control"
                      id="windowsClassicAppVersionInput"
                      placeholder="1.0.0"
                      required
                      v-model="windowsForm.classicPackage.version"
                    />
                  </div>
                </div>

              </div>

              <div class="form-group">
                <label for="windowsStartUrlInput">
                  URL
                  <i
                    class="fas fa-info-circle"
                    title="This is the URL for your PWA."
                    aria-label="This is the URL for your PWA."
                    role="definition"
                  ></i>
                </label>
                <input
                  type="url"
                  class="form-control"
                  id="windowsStartUrlInput"
                  placeholder="/index.html"
                  required
                  v-model="windowsForm.url"
                />
              </div>

              <div class="form-group">
                <label for="windowsManifestUrlInput">
                  Manifest URL
                  <i
                    class="fas fa-info-circle"
                    title="The URL to your app manifest."
                    aria-label="The URL to your app manifest."
                    role="definition"
                  ></i>
                </label>
                <input
                  type="url"
                  class="form-control"
                  id="windowsManifestUrlInput"
                  placeholder="https://mysite.com/manifest.json"
                  required
                  v-model="windowsForm.manifestUrl"
                />
              </div>

            </div>

            <!-- right half of the options dialog -->
            <div class="col-lg-6 col-md-12">
              <div class="form-group">
                <label for="windowsIconUrlInput">
                  <a href="https://github.com/pwa-builder/pwabuilder-windows-chromium-docs/blob/master/image-recommendations.md" target="_blank">
                    Icon URL
                    <i
                      class="fas fa-info-circle"
                      title="A large, square, PNG image from which PWABuilder will generate all required Windows app icons. Should be 512x512 or larger. Click to learn more."
                      aria-label="A large, square, PNG image from which PWABuilder will generate all required Windows app icons. Should be 512x512 or larger. Click to learn more."
                      role="definition"
                    ></i>
                  </a>
                </label>
                <input
                  type="url"
                  class="form-control"
                  id="windowsIconUrlInput"
                  placeholder="https://myawesomepwa.com/512x512.png"
                  v-model="windowsForm.images.baseImage"
                />
              </div>

              <div class="form-group">
                  <label for="windowsDisplayNameInput">
                    <a target="_blank" href="https://github.com/pwa-builder/pwabuilder-windows-chromium-docs/blob/master/find-publisher.md">
                      Publisher display name
                      <i
                        class="fas fa-info-circle"
                        title="The display name of your app's publisher. You can find this in Windows Partner Center. Click to learn more."
                        aria-label="The display name of your app's publisher. You can find this in Windows Partner Center. Click to learn more."
                        role="definition"
                      ></i>
                    </a>
                  </label>
                  <input
                    type="text"
                    class="form-control"
                    for="windowsDisplayNameInput"
                    required
                    placeholder="US"
                    v-model="windowsForm.publisher.displayName"
                  />
                </div>

                <div class="form-group">
                  <label for="windowsPublisherIdInput">
                    <a target="_blank" href="https://github.com/pwa-builder/pwabuilder-windows-chromium-docs/blob/master/find-publisher.md">
                      Publisher ID
                      <i
                        class="fas fa-info-circle"
                        title="Your Windows Publisher ID. You can find this value in Windows Partner Center. Click to learn more."
                        aria-label="Your Windows Publisher ID. You can find this value in Windows Partner Center. Click to learn more."
                        role="definition"
                      ></i>
                    </a>
                  </label>
                  <input
                    type="text"
                    class="form-control"
                    id="windowsPublisherIdInput"
                    required
                    placeholder="CN=3a54a224-05dd-42aa-85bd-3f3c1478fdca"
                    v-model="windowsForm.publisher.commonName"
                  />
                </div>
            </div>
          </div>
        </form>
      </section>
    </Modal>

    <div v-if="openAndroid" ref="androidModal" id="androidPlatModal">
      <button @click="closeAndroidModal()" id="closeAndroidPlatButton">
        <i class="fas fa-times"></i>
      </button>

      <section class="androidModalBody">
        <div>
          <p class="androidModalP">
            Download your
            <a
              href="https://developers.google.com/web/updates/2019/08/twas-quickstart"
            >PWA package</a>
            for Google Play
          </p>

          <p>
            Your download will contain
            <a
              href="https://github.com/pwa-builder/CloudAPK/blob/master/Next-steps.md"
            >instructions</a>
            for submitting your app to the Google Play store.
          </p>

          <p v-if="this.androidForm.package_name">
            <span>Package Name:</span>
            {{ $t(this.androidForm.package_name) }}
          </p>
        </div>

        <div id="androidModalButtonSection">
          <Download
            :showMessage="true"
            :androidOptions="this.androidForm"
            id="androidDownloadButton"
            class="androidDownloadButton"
            platform="androidTWA"
            message="Download"
            v-on:apkDownloaded="showInstall($event)"
            v-on:downloadPackageError="showPackageDownloadError($event)"
          />
          <button class="androidDownloadButton" @click="openAndroidOptionModal()">Options</button>
        </div>

        <!-- justin revisit -->
        <!--<div id="androidInstallWrapper" v-if="apkDownloaded">
          <p>Quickly install your APK to your device for testing! All you need to do is plug in your device via a USB cord.</p>

          <div id="androidInstallActions">
            <button class="androidDownloadButton" @click="connectDevice()">Connect to Device</button>
            <button
              :disabled="!connected"
              class="androidInstallButton"
              @click="installAPK()"
            >
            <span v-if="!installing">Install APK</span>

              <div v-if="installing">
                <div id="installSpinner" v-if="!isReady">
                  <div class="flavor">
                    <div class="colorbands"></div>
                  </div>
                  <div class="icon">
                    <div class="lds-dual-ring"></div>
                  </div>
                </div>
              </div>
            </button>
          </div>
        </div>-->

        <div id="extraSection">
          <p>
            Your PWA will be a <a href="https://developers.google.com/web/updates/2019/08/twas-quickstart" target="_blank" rel="noopener">Trusted Web Activity</a>.
            <Download
              :showMessage="true"
              id="legacyDownloadButton"
              class="webviewButton"
              platform="android"
              message="Use a legacy webview instead"
              v-on:downloadPackageError="showPackageDownloadError($event)"
            />
          </p>
        </div>
      </section>
    </div>

    <div v-if="openAppGallery" ref="appGalleryModal" id="appGalleryPlatModal">
      <button @click="closeAppGalleryModal()" id="closeAppGalleryPlatButton">
        <i class="fas fa-times"></i>
      </button>

      <section id="appGalleryModalBody">
        <div style="width:100%;">
          <form-wizard
            title="AppGallery + HMS Download, Publish Options"
            subtitle=""
            stepSize="sm"
            color="#9337d8"
            @on-loading="clearError"
            finish-button-text="Close"
            @on-complete="closeAppGalleryModal()"
            >
            <tab-content title="APK options" :before-change="validateAPKOptions">
              <div class="l-generator-error" v-if="appGalleryPWAError">
                <span v-for="err in appGalleryPWAError"><span class="icon-exclamation"></span> {{err}}</br></span>
              </div>
              <div class="l-generator-error" v-if="appGalleryPWAWarning">
                <span v-for="err in appGalleryPWAWarning"><span class="icon-exclamation"></span> {{err}}</br></span>
              </div>
              <p style="width:100%;">Step 1: Customize your Android package below</p>

              <section id="appGalleryModalBody" class="appGalleryOptionsModalBody">
                <form style="width: 100%">
                  <div class="row">
                    <div class="col-lg-6 col-md-12">
                      <div class="form-group">
                        <label for="packageIdInput">
                          {{ $t("publish.label_package_name") }}
                          <i
                            class="fas fa-info-circle"
                            title="The unique identifier of your app. It should contain only letters, numbers, and periods. Example: com.companyname.appname"
                            aria-label="The unique identifier of your app. It should contain only letters, numbers, and periods. Example: com.companyname.appname"
                            role="definition"
                          ></i>
                        </label>
                        <input
                          id="packageIdInput"
                          class="form-control"
                          :placeholder="$t('publish.placeholder_package_name')"
                          type="text"
                          required
                          v-model="appGalleryForm.packageId"
                        />
                      </div>

                      <div class="row">
                        <div class="col-lg-6 col-md-12">
                          <div class="form-group">
                            <label for="appNameInput">App name</label>
                            <input
                              type="text"
                              class="form-control"
                              id="appNameInput"
                              placeholder="My Awesome PWA"
                              required
                              v-model="appGalleryForm.name"
                            />
                          </div>
                        </div>

                        <div class="col-lg-6 col-md-12">
                          <div class="form-group">
                            <label for="appLauncherNameInput">
                              Launcher name
                              <i
                                class="fas fa-info-circle"
                                title="The app name used on the Android launch screen. Typically, this is the short name of the app."
                                aria-label="The app name used on the Android launch screen. Typically, this is the short name of the app."
                                role="definition"
                              ></i>
                            </label>
                            <input
                              type="text"
                              class="form-control"
                              id="appLauncherNameInput"
                              placeholder="Awesome PWA"
                              required
                              v-model="appGalleryForm.launcherName"
                            />
                          </div>
                        </div>
                      </div>

                      <div class="row">
                        <div class="col-lg-6 col-md-12">
                          <div class="form-group">
                            <label for="appVersionInput">
                              App version
                              <i
                                class="fas fa-info-circle"
                                title="The version of your app displayed to users. This is a string, typically in the form of '1.0.0.0'. Maps to android:versionName."
                                aria-label
                                role="definition"
                              ></i>
                            </label>
                            <input
                              type="text"
                              class="form-control"
                              id="appVersionInput"
                              placeholder="1.0.0.0"
                              required
                              v-model="appGalleryForm.appVersion"
                            />
                          </div>
                        </div>

                        <div class="col-lg-6 col-md-12">
                          <div class="form-group">
                            <label for="appVersionCodeInput">
                              <a
                                href="https://developer.android.com/studio/publish/versioning#appversioning"
                                target="_blank"
                                rel="noopener"
                              >App version code</a>
                              <i
                                class="fas fa-info-circle"
                                title="A positive integer used as an internal version number. This is not shown to users. Android uses this value to protect against downgrades. Maps to android:versionCode."
                                aria-label="A positive integer used as an internal version number. This is not shown to users. Android uses this value to protect against downgrades. Maps to android:versionCode."
                                role="definition"
                                style="margin-left: 5px;"
                              ></i>
                            </label>
                            <input
                              type="number"
                              min="1"
                              max="2100000000"
                              class="form-control"
                              id="appVersionCodeInput"
                              placeholder="1"
                              required
                              v-model="appGalleryForm.appVersionCode"
                            />
                          </div>
                        </div>
                      </div>

                      <div class="form-group">
                        <label for="hostInput">Host</label>
                        <input
                          type="url"
                          class="form-control"
                          id="hostInput"
                          placeholder="https://mysite.com"
                          required
                          v-model="appGalleryForm.host"
                        />
                      </div>

                      <div class="form-group">
                        <label for="startUrlInput">
                          Start URL
                          <i
                            class="fas fa-info-circle"
                            title="The start path for the TWA. Must be relative to the Host URL. You can specify '/' if you don't have a start URL different from Host."
                            aria-label="The start path for the TWA. Must be relative to the Host URL."
                            role="definition"
                          ></i>
                        </label>
                        <input
                          type="url"
                          class="form-control"
                          id="startUrlInput"
                          placeholder="/index.html"
                          required
                          v-model="appGalleryForm.startUrl"
                        />
                      </div>

                      <div class="form-group">
                        <label for="manifestUrlInput">Manifest URL</label>
                        <input
                          type="url"
                          class="form-control"
                          id="manifestUrlInput"
                          placeholder="https://mysite.com/manifest.json"
                          required
                          v-model="appGalleryForm.webManifestUrl"
                          @keyup="checkIconAndManifest()"
                        />
                      </div>

                      <div class="form-group">
                        <div v-if="appGalleryForm.iconUrl.length == 0" style="width:100%; display:block;">
                          <div style="width: 100%;">
                            <span style="float: left; width: 18%; font-size: 16px;">
                              Get Icon
                            </span>
                            <input type="text" v-model="getIconInput" style="font-size: 14px; width: 48%; float: left; margin-right: 4px;" />
                            <div class="getIconButton" @click="getIcon(getIconInput)">
                              Search
                            </div>
                          </div>
                          <br><br>
                          <div style="font-size: 10px; width: 100%;" v-show="getIconStatus">
                            Getting icons using App name(s) "{{ getIconInput }}".
                          </div>
                          <div style="color: red; font-size: 10px; line-height: 12px; width: 100%;" v-show="getIconFailed">
                            Getting icons using App name(s) "{{ getIconInput }}" failed. Please change the app name and try again. Tips: try to shorten the keywords and make it more relevant.
                          </div>
                          <div style="border: solid 1px #999; overflow-x: scroll; border-radius:4px; padding: 4px;" v-show="iconsArray.length > 0">
                            <ul style="padding:0px; margin:0px; white-space: nowrap;" v-for="icon in iconsArray">
                              <li style="margin-right:4px; border: solid 1px #cfcfcf; padding: 4px; margin: 4px; border-radius:4px; display: block; float:left; list-style-type: none;">
                                <img v-bind:src="icon.src" style="height: 30px;" @click="changeIconUrl(icon.src)" />
                              </li>
                            </ul>
                          </div>
                          <div style="font-size: 11px; color: #888; font-style: italic; width: 100%; display: block;" v-show="iconsArray.length > 0">
                            Click on the image to fill Icon URL field.
                          </div>
                        </div>
                        <label for="iconUrlInput">Icon URL <span v-show="appGalleryForm.iconUrl.length > 0" @click="clearIconUrl" style="border: solid 1px #cfcfcf; padding: 3px 6px; font-size: 10px; border-radius: 4px;">X</span>&nbsp;&nbsp;&nbsp;&nbsp;<img v-bind:src="appGalleryForm.iconUrl" style="height: 30px; top: 6px; position: relative;" /></label>
                        <input
                          type="url"
                          class="form-control"
                          id="iconUrlInput"
                          placeholder="https://myawesomepwa.com/512x512.png"
                          v-model="appGalleryForm.iconUrl"
                          @keyup="checkIconAndManifest()"
                        />
                      </div>

                      <div class="form-group">
                        <label for="maskIconUrlInput">
                          <a
                            href="https://web.dev/maskable-icon"
                            title="Read more about maskable icons"
                            target="_blank"
                            rel="noopener"
                            aria-label="Read more about maskable icons"
                          >Maskable icon</a> URL
                          <i
                            class="fas fa-info-circle"
                            title="The URL to an icon with a minimum safe zone of trimmable padding, enabling rounded icons on certain Android platforms. Optional."
                            aria-label="The URL to an icon with a minimum safe zone of trimmable padding, enabling rounded icons on certain Android platforms. Optional."
                            role="definition"
                          ></i>
                        </label>
                        <input
                          type="url"
                          class="form-control"
                          id="maskIconUrlInput"
                          placeholder="https://myawesomepwa.com/512x512-maskable.png"
                          v-model="appGalleryForm.maskableIconUrl"
                        />
                      </div>

                      <div class="form-group">
                        <label for="monochromeIconUrlInput">
                          <a
                            href="https://w3c.github.io/manifest/#monochrome-icons-and-solid-fills"
                            target="_blank"
                            rel="noopener"
                          >Monochrome icon</a> URL
                          <i
                            class="fas fa-info-circle"
                            title="The URL to an icon containing only white and black colors, enabling Android to fill the icon with user-specified color or gradient depending on theme, color mode, or contrast settings. Optional."
                            aria-label="The URL to an icon containing only white and black colors, enabling Android to fill the icon with user-specified color or gradient depending on theme, color mode, or contrast settings. Optional."
                            role="definition"
                          ></i>
                        </label>
                        <input
                          type="url"
                          class="form-control"
                          id="monochromeIconUrlInput"
                          placeholder="https://myawesomepwa.com/512x512-monochrome.png"
                          v-model="appGalleryForm.monochromeIconUrl"
                        />
                      </div>

                      <div class="row">
                        <div class="col-lg-4 col-md-12">
                          <div class="form-group">
                            <label for="themeColorInput">
                              Status bar color
                              <i
                                class="fas fa-info-circle"
                                title="Also known as the theme color, this is the color of the Android status bar in your app. Note: the status bar will be hidden if Display Mode is set to fullscreen."
                                aria-label="Also known as the theme color, this is the color of the Android status bar in your app. Note: the status bar will be hidden if Display Mode is set to fullscreen."
                                role="definition"
                              ></i>
                            </label>
                            <input
                              type="color"
                              class="form-control"
                              id="themeColorInput"
                              v-model="appGalleryForm.themeColor"
                            />
                          </div>
                        </div>

                        <div class="col-lg-4 col-md-12">
                          <div class="form-group">
                            <label for="navigationColorInput">
                              Nav bar color
                              <i
                                class="fas fa-info-circle"
                                title="The color of the Android navigation bar in your app. Note: the navigation bar will be hidden if Display Mode is set to fullscreen."
                                aria-label="The color of the Android navigation bar in your app. Note: the navigation bar will be hidden if Display Mode is set to fullscreen."
                                role="definition"
                              ></i>
                            </label>
                            <input
                              type="color"
                              class="form-control"
                              id="navigationColorInput"
                              v-model="appGalleryForm.navigationColor"
                            />
                          </div>
                        </div>

                        <div class="col-lg-4 col-md-12">
                          <div class="form-group">
                            <label for="bgColorInput">
                              Splash color
                              <i
                                class="fas fa-info-circle"
                                title="Also known as background color, this is the color of the splash screen for your app."
                                aria-label="Also known as background color, this is the color of the splash screen for your app."
                                role="definition"
                              ></i>
                            </label>
                            <input
                              type="color"
                              class="form-control"
                              id="bgColorInput"
                              v-model="appGalleryForm.backgroundColor"
                            />
                          </div>
                        </div>
                      </div>

                      <div class="row">
                        <div class="col-lg-12 col-md-12">
                          <div class="form-group">
                            <label for="appNameInput">
                              Whitelist 
                              <i
                                class="fas fa-info-circle"
                                title="List the URLs that is allowed for in-app redirection. Separated by commas."
                                aria-label="List the URLs that is allowed for in-app redirection. Separated by commas."
                                role="definition"
                              ></i>
                              <i>
                                (comma separated)
                              </i>
                            </label>
                            <input
                              type="text"
                              class="form-control"
                              id="whitelistInput"
                              placeholder=""
                              required
                              v-model="appGalleryForm.whitelist"
                            />
                          </div>
                        </div>
                      </div>

                    </div>
                    <div class="col-lg-6 col-md-12">
                      

                      <div class="form-group">
                        <label for="splashFadeoutInput">Splash screen fade out duration (ms)</label>
                        <input
                          type="number"
                          class="form-control"
                          id="splashFadeoutInput"
                          placeholder="300"
                          v-model="appGalleryForm.splashScreenFadeOutDuration"
                        />
                      </div>

                      <div class="form-group">
                        <label>Fallback behavior</label>
                        <div class="form-check">
                          <input
                            class="form-check-input"
                            type="radio"
                            name="fallbackType"
                            id="fallbackCustomTabsInput"
                            value="customtabs"
                            v-model="appGalleryForm.fallbackType"
                          />
                          <label class="form-check-label" for="fallbackCustomTabsInput">
                            Custom Tabs
                            <i
                              class="fas fa-info-circle"
                              title="Use Chrome Custom Tabs as a fallback for your PWA when the full trusted web activity (TWA) experience is unavailable."
                              aria-label="When trusted web activity (TWA) is unavailable, use Chrome Custom Tabs as a fallback for your PWA."
                              role="definition"
                            ></i>
                          </label>
                        </div>
                        <div class="form-check">
                          <input
                            class="form-check-input"
                            type="radio"
                            name="fallbackType"
                            id="fallbackWebViewInput"
                            value="webview"
                            v-model="appGalleryForm.fallbackType"
                          />
                          <label class="form-check-label" for="fallbackWebViewInput">
                            Web View
                            <i
                              class="fas fa-info-circle"
                              title="Use a web view as the fallback for your PWA when the full trusted web activity (TWA) experience is unavailable."
                              aria-label="When trusted web activity (TWA) is unavailable, use a web view as the fallback for your PWA."
                              role="definition"
                            ></i>
                          </label>
                        </div>
                      </div>

                      <div class="form-group">
                        <label>Display mode</label>
                        <div class="form-check">
                          <input
                            class="form-check-input"
                            type="radio"
                            name="displayMode"
                            id="standaloneDisplayModeInput"
                            value="standalone"
                            v-model="appGalleryForm.display"
                          />
                          <label class="form-check-label" for="standaloneDisplayModeInput">
                            Standalone
                            <i
                              class="fas fa-info-circle"
                              title="Your PWA will use the whole screen but keep the Android status bar and navigation bar."
                              aria-label="Your PWA will use the whole screen but keep the Android status bar and navigation bar."
                              role="definition"
                            ></i>
                          </label>
                        </div>
                        <div class="form-check">
                          <input
                            class="form-check-input"
                            type="radio"
                            name="displayMode"
                            id="fullscreenDisplayModeInput"
                            value="fullscreen"
                            v-model="appGalleryForm.display"
                          />
                          <label class="form-check-label" for="fullscreenDisplayModeInput">
                            Fullscreen
                            <i
                              class="fas fa-info-circle"
                              title="Your PWA will use the whole screen and remove the Android status bar and navigation bar. Suitable for immersive experiences such as games or media apps."
                              aria-label="Your PWA will use the whole screen and remove the Android status bar and navigation bar. Suitable for immersive experiences such as games or media apps."
                              role="definition"
                            ></i>
                          </label>
                        </div>
                      </div>

                      <div class="form-group">
                        <label>Notifications</label>
                        <div class="form-check">
                          <input
                            class="form-check-input"
                            type="checkbox"
                            id="enableNotificationsInput"
                            v-model="appGalleryForm.enableNotifications"
                          />
                          <label class="form-check-label" for="enableNotificationsInput">
                            Enable
                            <i
                              class="fas fa-info-circle"
                              title="Whether to enable Push Notification Delegation. If enabled, your PWA can send push notifications without browser permission prompts."
                              aria-label
                              role="definition"
                            ></i>
                          </label>
                        </div>
                      </div>

                      <div class="form-group">
                        <label>Signing key</label>
                        <div class="form-check">
                          <input class="form-check-input" type="radio" name="signingInput" id="generateSigningKeyInput" value="new" v-model="appGalleryForm.signingMode" @change="appGallerySigningModeChanged">
                          <label class="form-check-label" for="generateSigningKeyInput">
                            Create new
                            <i
                              class="fas fa-info-circle"
                              title="PWABuilder will generate a new signing key for you and sign your APK with it. Your download will contain the new signing key and passwords."
                              aria-label="PWABuilder will generate a new signing key for you and sign your APK with it. Your download will contain the new signing key and passwords."
                              role="definition"
                            ></i>
                          </label>
                        </div>
                        <!-- <div class="form-check">
                          <input class="form-check-input" type="radio" name="signingInput" id="unsignedInput" value="none" v-model="appGalleryForm.signingMode" @change="appGallerySigningModeChanged">
                          <label class="form-check-label" for="unsignedInput">
                            None
                            <i
                              class="fas fa-info-circle"
                              title="PWABuilder will generate an unsigned APK. Google Play Store will sign your package. This is Google's recommended approach."
                              aria-label="PWABuilder will generate an unsigned APK. Google Play Store will sign your package. This is Google's recommended approach."
                              role="definition"
                            ></i>
                          </label>
                        </div>
                        <div class="form-check">
                          <input class="form-check-input" type="radio" name="signingInput" id="useMySigningInput" value="mine" v-model="appGalleryForm.signingMode" @change="appGallerySigningModeChanged">
                          <label class="form-check-label" for="useMySigningInput">
                            Use mine
                            <i
                              class="fas fa-info-circle"
                              title="Upload your existing signing key. Use this option if you already have a signing key and you want to publish a new version of an existing app in Google Play."
                              aria-label="Upload your existing signing key. Use this option if you already have a signing key and you want to publish a new version of an existing app in Google Play."
                              role="definition"
                            ></i>
                          </label>
                        </div> -->
                      </div>

                      <div
                        v-if="appGalleryForm.signingMode === 'mine' || appGalleryForm.signingMode === 'new'"
                        style="margin-left: 15px;"
                      >
                        <div class="form-group" v-if="appGalleryForm.signingMode === 'mine'">
                          <label for="signingKeyInput">Key file</label>
                          <input
                            type="file"
                            class="form-control"
                            id="signingKeyInput"
                            @change="appGallerySigningKeyUploaded"
                            accept=".keystore"
                            required
                            style="border: none;"
                          />
                        </div>

                        <div class="form-group">
                          <label for="signingKeyAliasInput">Key alias</label>
                          <input
                            type="text"
                            class="form-control"
                            id="signingKeyAliasInput"
                            placeholder="my-key-alias"
                            required
                            v-model="appGalleryForm.signing.alias"
                          />
                        </div>

                        <div class="form-group">
                          <label for="signingKeyFullNameInput">Key full name</label>
                          <input
                            type="text"
                            class="form-control"
                            id="signingKeyFullNameInput"
                            required
                            placeholder="John Doe"
                            v-model="appGalleryForm.signing.fullName"
                            pattern="[a-zA-Z0-9\s]+"
                          />
                        </div>

                        <div class="form-group">
                          <label for="signingKeyOrgInput">Key organization</label>
                          <input
                            type="text"
                            class="form-control"
                            id="signingKeyOrgInput"
                            required
                            placeholder="My Company"
                            v-model="appGalleryForm.signing.organization"
                            pattern="[a-zA-Z0-9\s]+"
                          />
                        </div>

                        <div class="form-group">
                          <label for="signingKeyOrgUnitInput">Key organizational unit</label>
                          <input
                            type="text"
                            class="form-control"
                            id="signingKeyOrgUnitInput"
                            required
                            placeholder="Engineering Department"
                            v-model="appGalleryForm.signing.organizationalUnit"
                          />
                        </div>

                        <div class="form-group">
                          <label for="signingKeyCountryCodeInput">
                            Key country code
                            <i
                              class="fas fa-info-circle"
                              title="The 2 letter country code to list on the signing key"
                              aria-label="The 2 letter country code to list on the signing key"
                              role="definition"
                            ></i>
                          </label>
                          <input
                            type="text"
                            class="form-control"
                            id="signingKeyCountryCodeInput"
                            required
                            placeholder="US"
                            v-model="appGalleryForm.signing.countryCode"
                          />
                        </div>

                        <div class="form-group">
                          <label for="signingKeyPasswordInput">Key password</label>
                          <input type="password" class="form-control" id="signingKeyPasswordInput" v-model="appGalleryForm.signing.keyPassword" :placeholder="appGalleryPasswordPlaceholder" />
                        </div>

                        <div class="form-group">
                          <label for="signingKeyStorePasswordInput">Key store password</label>
                          <input type="password" class="form-control" id="signingKeyStorePasswordInput" v-model="appGalleryForm.signing.storePassword" :placeholder="appGalleryPasswordPlaceholder" />
                        </div>
                      </div>
                    </div>
                  </div>
                </form>
              </section>

            </tab-content>
            <tab-content title="HMS">
                <div style="width: 100%; height: 400px;">
                  <p style="width:100%;">Step 2: Select HMS kits to be included in your APK</p>
                  <div style="width: 210px; border:1px solid #333; height: 210px; border-radius:10%; float: left; padding: 10px; margin: 2%; position: relative;">
                    <div style="text-align:center; font-size: 20px; font-weight: bold; width: 100%;">HMS Push Kit</div>
                    <div style="width: 30px; height: 30px; z-index:3; position: relative; left: 140px; top: 10px;">
                      <input type="checkbox" id="hmsPushKit" name="hmsPushKit" v-model="appGalleryForm.hmsPushKit" @change="printAppGalleryForm()">
                    </div>
                    <div style="width: 110px; height: 110px; z-index:2; position: relative; margin: auto; top: -10px;">
                      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 139.88 154.66"><defs><style>.cls-1{fill:#f3a641;}.cls-2{fill:#fefefe;}.cls-3{fill:#fefbf8;}.cls-4{fill:#fdf9f3;}.cls-5{fill:#fdf8f2;}.cls-6{fill:#f3a742;}</style></defs><g id="Layer_2" data-name="Layer 2"><g id="Layer_1-2" data-name="Layer 1"><path class="cls-1" d="M139.53,47.63c0-6.05-4.36-13.45-9.62-16.44L79,2.21c-5.26-3-13.83-2.94-19,.12L9.43,31.91C4.21,35,0,42.41,0,48.46L.35,107c0,6,4.37,13.45,9.62,16.45l50.89,29c5.26,3,13.83,2.95,19-.11l50.54-29.58c5.23-3.06,9.47-10.51,9.43-16.56Z"/><path class="cls-2" d="M56.47,52.86c3,.41,6,0,8.87,1,2.22.82,4.21-.22,6.07-1.42,7.15-4.62,15-7.85,22.92-10.71a3.05,3.05,0,0,1,.46-.2c2.9-.51,6.42-1.92,8.57,0,2,1.79,0,5-.76,7.44-2.7,8.55-5.85,16.91-11.21,24.22A5.28,5.28,0,0,0,90.56,77c.15,1.82,0,3.74.49,5.45,2.6,8.35.18,14.82-6.7,20-2.44,1.85-4.44,4.29-6.65,6.47-1,1-2,2.4-3.78,1.84s-1.51-2.34-1.64-3.77c-.26-2.64-.38-5.3-.72-7.93-.83-6.34-.89-6.42-6.87-4.8A10.37,10.37,0,0,1,57.91,94c-6.87-2.7-9.65-8.06-7.4-16.2,1.12-4.08.12-5.2-3.72-4.8s-7.53-1-11.28-1.56c-2.6-.4-3-1.9-1.22-3.76,4.6-4.73,9.11-9.57,14-14C50.49,51.73,53.71,53,56.47,52.86Z"/><path class="cls-3" d="M33.91,112.07c-2.28-.68-2.28-2.49-.91-4.07,3-3.5,6.32-6.82,9.63-10.08,1.25-1.24,3.07-1.77,4.42-.24s.55,3-.81,4.33c-3.14,3-6.23,6-9.35,8.93A3.58,3.58,0,0,1,33.91,112.07Z"/><path class="cls-4" d="M57.78,104.55c-1.82,2.73-3.66,6-7.38,7.18-1.64.54-2.4-.59-2.32-2.25s5.47-7.44,7.32-7.61C57,101.73,58,102.38,57.78,104.55Z"/><path class="cls-5" d="M42.35,88.53c-.1,2-5.73,7.94-7.35,7.93-2.37,0-2.22-2-1.78-3.21,1.09-3,3.6-5.09,6.26-6.62C41,85.78,42.32,86.84,42.35,88.53Z"/><path class="cls-6" d="M82.1,69a7.12,7.12,0,0,1-7-6.43c-.14-3.31,3.14-6.9,6.56-6.71,3.67.2,6.34,2,6.56,6.06C88.42,65.86,85.78,68.88,82.1,69Z"/></g></g></svg>
                    </div>
                    <div style="width: 100%; height: 30px; position: relative; margin-top: 8px; left: 0px; font-size: 11px; text-align:center;">
                      <a href="https://developer.huawei.com/consumer/en/hms/huawei-pushkit" target="_new" title="Learn more about HMS Push Kit" >
                        Learn more
                      </a>
                    </div>
                  </div>
                  <div style="width: 210px; border:1px solid #333; height: 210px; border-radius:10%; float: left; padding: 10px; margin: 2%; position: relative;">
                    <div style="text-align:center; font-size: 20px; font-weight: bold; width: 100%;">HMS Analytics Kit</div>
                    <div style="width: 30px; height: 30px; z-index:3; position: relative; left: 140px; top: 10px;">
                      <input type="checkbox" id="hmsAnalyticsKit" name="hmsAnalyticsKit" v-model="appGalleryForm.hmsAnalyticsKit"  @change="printAppGalleryForm()">
                    </div>
                    <div style="width: 110px; height: 110px; z-index:2; position: relative; margin: auto; top: -10px;">
                      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 139.88 154.66"><defs><style>.cls-1{fill:#f3a641;}.cls-2{fill:#fff;}</style></defs><g id="Layer_2" data-name="Layer 2"><g id="Layer_1-2" data-name="Layer 1"><path class="cls-1" d="M139.53,47.63c0-6.05-4.36-13.45-9.62-16.44L79,2.21c-5.26-3-13.83-2.94-19,.12L9.43,31.91C4.21,35,0,42.41,0,48.46L.35,107c0,6,4.37,13.45,9.62,16.45l50.89,29c5.26,3,13.83,2.95,19-.11l50.54-29.58c5.23-3.06,9.47-10.51,9.43-16.56Z"/><path class="cls-2" d="M103.4,75.77a36.17,36.17,0,0,1-7.82,22l9.91,9.82c.88,1-4.23,6.18-5.3,5.3l-10.47-9.4a37.59,37.59,0,0,1-21.07,7.07A34.75,34.75,0,1,1,103.4,75.77Z"/><circle class="cls-1" cx="68.65" cy="75.77" r="30.79"/><circle class="cls-2" cx="68.65" cy="75.77" r="26.37"/><rect class="cls-1" x="55.96" y="68.09" width="4.74" height="19.6" rx="2"/><rect class="cls-1" x="66.22" y="60.82" width="4.74" height="26.87" rx="2"/><rect class="cls-1" x="76.49" y="72.72" width="4.74" height="14.98" rx="2"/></g></g></svg>
                    </div>
                    <div style="width: 100%; height: 30px; position: relative; margin-top: 8px; left: 0px; font-size: 11px; text-align:center;">
                      <a href="https://developer.huawei.com/consumer/en/hms/huawei-analyticskit" target="_new" title="Learn more about HMS Analytics Kit" >
                        Learn more
                      </a>
                    </div>
                  </div>
                  <div style="width: 210px; border:1px solid #333; height: 210px; border-radius:10%; float: left; padding: 10px; margin: 2%; position: relative;">
                    <div style="text-align:center; font-size: 20px; font-weight: bold; width: 100%;">HMS Ads Kit</div>
                    <div style="width: 30px; height: 30px; z-index:3; position: relative; left: 140px; top: 10px;">
                      <input type="checkbox" id="hmsAdsKit" name="hmsAdsKit" v-model="appGalleryForm.hmsAdsKit"  @change="printAppGalleryForm()">
                    </div>
                    <div style="width: 110px; height: 110px; z-index:2; position: relative; margin: auto; top: -10px;">
                      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 139.88 154.66"><defs><style>.cls-1{fill:#f3a641;}.cls-2{fill:#fff;}</style></defs><g id="Layer_2" data-name="Layer 2"><g id="Layer_1-2" data-name="Layer 1"><path class="cls-1" d="M139.53,47.63c0-6.05-4.36-13.45-9.62-16.44L79,2.21c-5.26-3-13.83-2.94-19,.12L9.43,31.91C4.21,35,0,42.41,0,48.46L.35,107c0,6,4.37,13.45,9.62,16.45l50.89,29c5.26,3,13.83,2.95,19-.11l50.54-29.58c5.23-3.06,9.47-10.51,9.43-16.56Z"/><polygon class="cls-2" points="70.98 37.9 28.37 105.81 113.6 105.81 70.98 37.9"/><polygon class="cls-1" points="70.98 60.91 42.81 105.81 99.16 105.81 70.98 60.91"/><polygon class="cls-2" points="70.98 80.53 55.12 105.81 86.85 105.81 70.98 80.53"/></g></g></svg>
                    </div>
                    <div style="width: 100%; height: 30px; position: relative; margin-top: 8px; left: 0px; font-size: 11px; text-align:center;">
                      <a href="https://developer.huawei.com/consumer/en/hms/huawei-adskit" target="_new" title="Learn more about HMS Ads Kit" >
                        Learn more
                      </a>
                    </div>
                  </div>
                  <div style="width: 100%; float: left;">
                    <p class="appGalleryText">What is <b>HMS</b>?</br>
                    <b>HMS (Huawei Mobile Services)</b> offers a rich array of open device and cloud capabilities, which facilitate efficient development, fast growth, and flexible monetization. This enables global developers to pursue groundbreaking innovation, deliver next-level user experiences, and make premium content and services broadly accessible.</br>
                    <a href="https://developer.huawei.com/consumer/en/hms" target="_new"><b>Click here</b></a> to more about HMS capabilities.</p>
                  </div>
                </div>

            </tab-content>
            <tab-content title="HMS+AppGallery Configs" :before-change="validateAGCHMS">
              <div style="width: 100%; height: 450px;">
                <div class="l-generator-error" v-if="appGalleryPWAError">
                  <span v-for="err in appGalleryPWAError"><span class="icon-exclamation"></span> {{err}}</br></span>
                </div>
                <p style="width:100%;">Step 3: Add agconnect-services.json <span v-show="appGalleryForm.hmsAdsKit">& configure HMS kit</span></p>

                <h2 v-show="appGalleryForm.hmsAdsKit">HMS Ads Kit</h2>
                <table width="100%" v-show="appGalleryForm.hmsAdsKit">
                  <tr>
                    <td style="width: 20px; height:20px;"> <input type="checkbox" id="hmsAdsSplash" name="hmsAdsSplash" v-model="appGalleryForm.hmsAdsSplash"></td>
                    <td>Splash</td>
                    <td><input type="text" placeholder="Ad ID from PPS" v-model="appGalleryForm.hmsAdsSplashId" @keydown="checkHMSAds(1)" maxlength="14" /></td>
                  </tr>
                  <tr>
                    <td style="width: 20px; height:20px;"> <input type="checkbox" id="hmsAdsTopBanner" name="hmsAdsTopBanner" v-model="appGalleryForm.hmsAdsTopBanner"></td>
                    <td>Top banner</td>
                    <td><input type="text" placeholder="Ad ID from PPS" v-model="appGalleryForm.hmsAdsTopBannerId" v-on:input="checkHMSAds(2)" maxlength="14" /></td>
                  </tr>
                  <tr>
                    <td style="width: 20px; height:20px;"> <input type="checkbox" id="hmsAdsBottomBanner" name="hmsAdsBottomBanner" v-model="appGalleryForm.hmsAdsBottomBanner" /></td>
                    <td>Bottom banner</td>
                    <td><input type="text" placeholder="Ad ID from PPS" v-model="appGalleryForm.hmsAdsBottomBannerId" v-on:input="checkHMSAds(3)" maxlength="14" /></td>
                  </tr>
                </table>
                <p class="appGalleryText" v-show="appGalleryForm.hmsAdsKit">How do I get the HMS Ads Kit IDs?</br>
                You need to create the HMS Ads Kit IDs through HUAWEI Ads Publisher Service, please <a href="https://developer.huawei.com/consumer/en/doc/development/HMSCore-Guides/publisher-service-introduction-0000001050064960" target="_new"><b>follow the instructions here</b></a> to generate it.</p>
                <hr v-show="appGalleryForm.hmsAdsKit"/>
                <h5>Select agconnect-services.json from your local directory</h5>
                <input type="file" name="agcs" accept="application/json" @change="onAGCSFileChange" class="form-control-file" />
                <hr />
                <p class="appGalleryText">What is <b>agconnect-services.json</b>?</br>
                <b>Agconnect-services.json</b> is a JSON document created by Huawei AppGallery Connect to allow your app to connect to their HMS and AppGallery Connect (AGC) backend API. You need to provide this file to enable HMS kit &amp; AGC capability. <a href="https://developer.huawei.com/consumer/en/doc/development/AppGallery-connect-Guides/agc-get-started" target="_new"><b>Click here</b></a> for step-by-step instructions to create agconnect-services.json for your AppGallery app.</p>
              </div>
            </tab-content>
            <tab-content title="Download/Publish">
              <p style="width:100%;">Step 4: Download or Publish (Optional) APK</p>
              <div>
                <vue-tabs>
                  <v-tab title="Download">
                    <br /><br /><br /><br />
                    <div>
                      <Download
                        :showMessage="true"
                        :androidOptions="JSON.stringify(this.appGalleryForm)"
                        id="appGalleryDownloadButton"
                        class="appGalleryDownloadButton"
                        platform="appgallery"
                        message="Build & Download My PWA"
                        v-on:apkDownloaded="showInstall($event)"
                      />
                    </div>
                    <br />
                    <p class="appGalleryDownloadText">Your download will contain instructions for submitting your app to HUAWEI AppGallery app store.</p>
                    <br /><br /><br /><br /><br /><br />
                  </v-tab>
                  <v-tab title="Publish">
                    <table width="100%">
                      <tr>
                        <td>
                          <label class="form-group">Client ID</label>
                          <input type="text" name="agcClientId" v-model="appGalleryPublish.client_id" class="form-control">
                          <br>
                        </td>
                        <td width="40"></td>
                        <td>
                          <label class="form-group">Client Key</label>
                          <input type="text" name="agcClientKey" v-model="appGalleryPublish.client_key" class="form-control">
                        </td>
                      </tr>
                      <tr>
                        <td>
                          <label class="form-group">App ID</label>
                          <input type="text" name="agcAppId" v-model="appGalleryPublish.app_id" class="form-control">
                        </td>
                        <td width="40"></td>
                        <td>
                          <h5>Select APK from your local directory</h5>
                          <input class="form-control-file" type="file" v-on:change="getAGAPK" accept=".apk" />Browse</button>
                        </td>
                      </tr>
                      <tr>
                        <td colspan="3">
                          <p class="appGalleryText">What is the purpose of this Publish interface? (optional)</br>
                          The Publish interface provides you a quick and easy option to upload and publish your PWA APK to your AppGallery developer account. To use this tool, please make sure that you have <a href="https://developer.huawei.com/consumer/en/doc/agc-create_app" target="_new"><b>created your app</b></a> to get your <b>App ID</b>, <a href="https://developer.huawei.com/consumer/en/doc/development/AppGallery-connect-Guides/agcapi-getstarted" target="_new"><b>configure your AppGallery Connect API</b></a> to get your <b>Client ID</b> &amp; <b>Client Key</b>. You do not need to use this interface to publish to your AppGallery account.</br>
                          More information:
                          <a href="https://developer.huawei.com/consumer/en/doc/start/registration-and-verification-0000001053628148" target="_new"><b>Get started as a Huawei Developer</b></a> |
                          <a href="https://developer.huawei.com/consumer/en/doc/distribution/app/agc-release_app" target="_new"><b>Publish your app in AppGallery.</b></a></br>
                          </p>
                          <Publish
                            :showMessage="true"
                            :appGalleryPublishAPK="this.appGalleryPublish"
                            class="appGalleryDownloadButton"
                            message="Publish APK to AppGallery"
                            aria-label="Publish APK to AppGallery"
                            v-on:appGalleryPublishSuccess="showAppGalleryPublishSuccess($event)"
                          />
                        </td>
                      </tr>
                    </table>
                  </v-tab>
                </vue-tabs>
              </div>
            </tab-content>
          </form-wizard>
          <!-- <p>
            Your download will contain
            <a href="https://github.com/pwa-builder/CloudAPK/blob/master/Next-steps.md">instructions</a>
            for submitting your app to the AppGallery.
          </p>

          <p v-if="this.androidForm.package_name">
            <span>Package Name:</span>
            {{ $t(this.androidForm.package_name) }}
          </p> -->
        </div>

        <!-- <div id="appGalleryModalButtonSection">
          <Download
            :showMessage="true"
            :androidOptions="this.androidForm"
            id="appGalleryDownloadButton"
            class="appGalleryDownloadButton"
            platform="androidTWA"
            message="Download"
            v-on:apkDownloaded="showInstall($event)"
            v-on:downloadPackageError="showPackageDownloadError($event)"
          />
          <button class="appGalleryDownloadButton" @click="openAppGalleryOptionModal()">Options</button>
        </div> -->

        <div id="extraSection">
          <p>
            Your PWA will be a Trusted Web Activity.
            <Download
              :showMessage="true"
              id="legacyDownloadButton"
              class="webviewButton"
              platform="android"
              message="Use a legacy webview instead (not recommended)"
              v-on:downloadPackageError="showPackageDownloadError($event)"
            />
          </p>
        </div>
      </section>
    </div>

    <div v-if="openWindows" ref="windowsModal" id="androidPlatModal">
      <button @click="closeAndroidModal()" id="closeAndroidPlatButton">
        <i class="fas fa-times"></i>
      </button>

      <section class="androidModalBody">
        <div>
          <p class="androidModalP">
            Download your PWA package for Microsoft Store
          </p>
          <p>
            Your download will contain <a href="https://github.com/pwa-builder/pwabuilder-windows-chromium-docs/blob/master/next-steps-edgehtml.md" target="_blank">instructions</a> for submitting your app to the Microsoft Store.
            Your app will be <a href="https://link.medium.com/7lXJkhtaKab" target="_blank" rel="noopener">powered by Chromium-based Edge</a> platform (preview).
          </p>
        </div>

        <div id="androidModalButtonSection" class="edgeBlock">
          <Download
              class="androidDownloadButton"
              platform="windows10new"
              :windowsOptions="this.windowsAnaheimForm"
              :message="'Download'"
              :showMessage="true"
              v-on:downloadPackageError="showPackageDownloadError($event)"
            />
          <button
            class="androidDownloadButton"
            @click="openWindowsOptionsModal('anaheim')">
            Options
          </button>
        </div>

        <div id="extraSection">
          <p>
            Use EdgeHTML (legacy Edge) instead:

            <Download
              id="legacyEdgeBetaDownloadButton"
              class="webviewButton"
              platform="windows10"
              :windowsOptions="this.windowsSpartanForm"
              :message="$t('publish.download')"
              :showMessage="true"
              v-on:downloadPackageError="showPackageDownloadError($event)" />

            <button class="legacyEdgeBetaOptionsButton" @click="openWindowsOptionsModal('spartan')">Options</button>
          </p>
        </div>
      </section>
    </div>

    <div v-if="openTeams" ref="teamsModal" id="teamsModal" class="platModal">
      <button @click="closeTeamsModal()" class="closeModalPlatButton">
        <i class="fas fa-times"></i>
      </button>
      <section class="platModalBody platModalForm">
        <div class="platModalField">
          <label for="package-name">
            <h4>Publisher Name</h4>
          </label>
          <input
            type="text"
            name="package-name"
            placeholder="Publisher Name"
            v-model="teamsForm.publisherName"
            @change="validateTeamsForm()"
          />
        </div>
        <div class="platModalField">
          <label for="short-description">
            <h4>Short app description</h4>
            <p>Describe your app in 80 characters or less</p>
          </label>
          <textarea
            name="short-description"
            class="short"
            :class="{
              error:
                teamsForm.shortDescription !== null &&
                teamsForm.shortDescription.length > 80,
            }"
            v-model="teamsForm.shortDescription"
            @change="validateTeamsForm()"
          ></textarea>
        </div>
        <div class="platModalField">
          <label for="long-description">
            <h4>Long app description</h4>
            <p>Describe your app in 4000 characters or less</p>
          </label>
          <textarea
            name="long-description"
            :class="{
              error:
                teamsForm.longDescription !== null &&
                teamsForm.longDescription.length > 4000,
            }"
            v-model="teamsForm.longDescription"
            @change="validateTeamsForm()"
          ></textarea>
        </div>
        <div class="platModalField">
          <label for="privacy">
            <h4>Privacy URL</h4>
          </label>
          <input
            name="privacy"
            type="text"
            placeholder="www.somewebsite/privacy"
            v-model="teamsForm.privacyUrl"
            @change="validateTeamsForm()"
          />
        </div>
        <div class="platModalField">
          <label for="terms">
            <h4>Terms of Use URL</h4>
          </label>
          <input
            names="terms"
            type="text"
            placeholder="www.somewebsite/termsofuse"
            v-model="teamsForm.termsOfUseUrl"
            @change="validateTeamsForm()"
          />
        </div>

        <div class="platModalField file-chooser">
          <label for="upload-image-color">
            <h4>App Image</h4>
            <p>
              The image needs to be 192x192, a solid background color,
              preferably the same as your w3c manifest background color.
            </p>
          </label>
          <button
            id="uploadIconImage-color"
            name="upload-image-color"
            @click="clickUploadColorFileInput()"
          >Choose File</button>
          <input
            id="upload-file-input-color"
            name="upload-image-color"
            type="file"
            accept="image/jpeg image/png image/svg+xml"
            @change="handleUploadColorIcon()"
          />
          <Loading
            :active="true"
            class="image-upload-loader"
            v-show="this.uploadColorLoaderActive"
          />
          <p class="file-description" v-show="!this.uploadColorLoaderActive">
            {{
            this.teamsForm.colorImageFile
            ? this.teamsForm.colorImageFile.name
            : "No file chosen"
            }}
          </p>
        </div>

        <div class="platModalField file-chooser">
          <label for="upload-image-outline">
            <h4>Teams Silhouette (optional)</h4>
            <p>
              This image needs to be 32x32, the background transparent, and the
              silhouette of your app icon in white.
            </p>
          </label>
          <button
            id="uploadIconImage-outline"
            name="upload-image-outline"
            @click="clickUploadOutlineFileInput()"
          >Choose File</button>
          <input
            id="upload-file-input-outline"
            name="upload-image-outline"
            type="file"
            accept="image/jpeg image/png image/svg+xml"
            @change="handleUploadOutlineIcon()"
          />
          <Loading
            :active="true"
            class="image-upload-loader"
            v-show="this.uploadOutlineLoaderActive"
          />
          <p class="file-description" v-show="!this.uploadOutlineLoaderActive">
            {{
            this.teamsForm.outlineImageFile
            ? this.teamsForm.outlineImageFile.name
            : "No file chosen"
            }}
          </p>
        </div>

        <div class="platModalButtonSection">
          <Download
            class="platModalDownloadButton"
            platform="msteams"
            :packageName="this.teamsForm.publisherName"
            :parameters="[JSON.stringify(this.teamsForm)]"
            :message="$t('publish.download')"
            :showMessage="true"
            v-on:downloadPackageError="showPackageDownloadError($event)"
          />
        </div>
      </section>
    </div>

    <section id="publishSideBySide">
      <section id="publishLeftSide">
        <div id="introContainer">
          <h2>Everything you need to build and publish PWA</h2>

          <p>
            Publish your PWA to app stores to make your app more discoverable to users.
          </p>

          <!--<div id="publishActionsContainer">-->
          <!--<button id="downloadAllButton">Download your PWA files</button>-->
          <!--<Download id="downloadAllButton" platform="web" message="Download your PWA files"/>-->
          <!--</div>-->

          <!--temp impl for demo-->
        </div>
      </section>

      <section id="publishRightSide">
        <div id="platformsListContainer" v-if="manifest">
          <ul>
            <div
              @mouseover="platCardHover($event)"
              @mouseleave="platCardUnHover($event)"
              id="pwaMainCard"
              class="pwaCard"
            >
              <div class="pwaCardHeaderBlock">
                <div class="pwaCardIconBlock">
                  <img class="pwaIcon" src="~/assets/images/pwaLogo.svg" alt="PWA Logo" />
                  <h2>Progressive Web App</h2>
                </div>
              </div>

              <p>
                If you haven’t already, download these files and publish it to
                your website. Making these changes to your website is all you
                need to become a PWA.
              </p>

              <section class="platformDownloadBar">
                <Download
                  class="platformDownloadButton"
                  platform="web"
                  message="Download your PWA files"
                  aria-label="Download your PWA files"
                  v-on:downloadPackageError="showPackageDownloadError($event)"
                />
              </section>
            </div>

            <div
              @mouseover="platCardHover($event)"
              @mouseleave="platCardUnHover($event)"
              id="pwaAndroidCard"
              class="pwaCard"
            >
              <div class="pwaCardHeaderBlock">
                <i class="fab fa-android platformIcon" aria-label="Android Icon"></i>
                <h2>Android</h2>
              </div>

              <p>
                Publish your PWA to the Google Play Store to make your app more discoverable for Android users.
              </p>

              <section class="platformDownloadBar">
                <button
                  class="platformDownloadButton"
                  @click="openAndroidModal()"
                  aria-label="Open Android Modal"
                >
                  <i class="fas fa-long-arrow-alt-down" aria-label="Open Android Icon"></i>
                </button>
              </section>
            </div>

            <!-- huawei platform -->
            <div
              @mouseover="platCardHover($event)"
              @mouseleave="platCardUnHover($event)"
              id="pwaAndroidCard"
              class="pwaCard"
            >
              <div class="pwaCardHeaderBlock">
                <svg
                  width="46"
                  height="30"
                  viewBox="0 0 38 30"
                  fill="none"
                  xmlns="http://www.w3.org/2000/svg"
                  aria-label="Huawei Icon"
                >
                  <style type="text/css">
                    .st0{fill:#020000;}
                  </style>
                  <path class="st0" d="M16.8,22.4c0.1-0.1,0.1-0.1,0.1-0.1c-2.7-5.8-6.1-11.2-10-16.2c0,0-3.1,3-2.9,5.9c0.1,1.4,0.8,2.7,1.8,3.6
                    c2.7,2.7,9.4,6.1,10.9,6.8C16.7,22.5,16.7,22.5,16.8,22.4 M15.7,24.7c0-0.1-0.1-0.1-0.2-0.1l0,0L4.6,25c1.2,2.1,3.2,3.8,5.3,3.3
                    c1.5-0.3,4.7-2.6,5.8-3.4l0,0C15.8,24.8,15.7,24.7,15.7,24.7 M15.9,23.7c0.1-0.1-0.1-0.2-0.1-0.2l0,0C11,20.3,1.7,15.3,1.7,15.3
                    c-1.1,3.3,0.6,6.8,3.8,8.1c0.7,0.3,1.4,0.4,2.1,0.4C7.7,23.8,14.1,23.8,15.9,23.7C15.8,23.8,15.9,23.8,15.9,23.7 M16.6,1.4
                    c-0.5,0.1-1.8,0.3-1.8,0.3c-1.7,0.4-3.1,1.7-3.6,3.4c-0.3,1.1-0.3,2.4,0,3.5c1,4.3,5.8,11.4,6.8,12.9c0.1,0.1,0.1,0.1,0.1,0.1
                    c0.1,0,0.1-0.1,0.1-0.1l0,0C19.9,5.6,16.6,1.4,16.6,1.4 M20.3,21.5c0.1,0,0.1,0,0.2-0.1l0,0c1.1-1.5,5.8-8.6,6.8-12.9
                    c0.3-1.1,0.3-2.4,0-3.5c-0.6-1.7-1.9-3-3.6-3.4c0,0-0.8-0.2-1.7-0.3c0,0-3.3,4.2-1.7,20.1l0,0C20.2,21.5,20.3,21.5,20.3,21.5
                     M22.9,24.6c0,0-0.1,0-0.1,0.1c0,0.1,0,0.1,0.1,0.2l0,0c1.1,0.7,4.3,3,5.8,3.4c0,0,2.9,1,5.3-3.3L22.9,24.6L22.9,24.6z M36.9,15.3
                    c0,0-9.4,5-14.2,8.3l0,0c-0.1,0.1-0.1,0.1-0.1,0.2c0,0,0.1,0.1,0.1,0.1l0,0h8.5c0.7-0.1,1.3-0.2,1.9-0.4c1.6-0.6,2.9-1.8,3.5-3.4
                    C37.3,18.5,37.4,16.8,36.9,15.3 M21.8,22.4c0.1,0.1,0.1,0.1,0.2,0l0,0c1.6-0.8,8.1-4.1,10.9-6.8c1.1-0.9,1.7-2.2,1.8-3.6
                    c0.2-3.1-2.9-5.9-2.9-5.9c-3.9,5-7.3,10.4-10,16.1l0,0C21.7,22.3,21.7,22.4,21.8,22.4"/>
                </svg>

                <h2>HUAWEI AppGallery</h2>
              </div>

              <p>
                PWAs are available through the browser on HUAWEI EMUI Phone, however your
                PWA can also be submitted to HUAWEI AppGallery by submitting the
                package you get below.
              </p>

              <section class="platformDownloadBar">
                <button
                  class="platformDownloadButton"
                  @click="openAppGalleryModal()"
                  aria-label="Open Android Modal"
                >
                  <i class="fas fa-long-arrow-alt-down" aria-label="Open Android Icon"></i>
                </button>
              </section>
            </div>

            <!--samsung platform-->
            <div
              @mouseover="platCardHover($event)"
              @mouseleave="platCardUnHover($event)"
              id="pwaSamsungCard"
              class="pwaCard"
            >
              <div class="pwaCardHeaderBlock">
                <svg
                  width="89"
                  height="30"
                  viewBox="0 0 89 30"
                  fill="none"
                  xmlns="http://www.w3.org/2000/svg"
                  aria-label="Samsung Icon"
                >
                  <path
                    d="M88.5919 7.15122C87.3559 0.0897582 66.5652 -2.11414 42.1107 2.2037C31.8699 4.04778 22.6001 6.74643 15.3609 9.75992C16.4644 9.8049 17.3031 10.0298 17.7887 10.5695C18.186 10.9743 18.3625 11.514 18.3625 12.1887V12.9083H15.9789V12.2787C15.9789 11.7839 15.6699 11.4241 15.1402 11.4241C14.6988 11.4241 14.3898 11.649 14.3015 12.0538C14.2574 12.1887 14.2574 12.3686 14.3015 12.5485C14.5663 13.628 18.0977 14.2577 18.4949 16.2367C18.5391 16.5065 18.6274 17.0463 18.4949 17.8109C18.2742 19.3851 16.9058 20.0148 15.1402 20.0148C12.7124 20.0148 11.6971 18.8454 11.6971 17.2262V16.4616H14.2574V17.4061C14.2574 17.9458 14.6546 18.2607 15.1402 18.2607C15.6257 18.2607 15.9347 18.0358 16.023 17.631C16.0672 17.4511 16.1113 17.1812 16.023 16.9563C15.5375 15.7419 12.2268 15.1572 11.8296 13.2232C11.7413 12.7734 11.7413 12.4136 11.7854 11.9188C11.8296 11.649 11.9178 11.4241 12.0061 11.2442C4.10478 15.0223 -0.574232 19.2052 0.0437503 22.8484C1.27972 29.9098 22.0704 32.1137 46.4807 27.7509C57.2072 25.8619 66.8742 22.9833 74.2458 19.7899C74.1575 19.7899 74.0251 19.7899 73.9368 19.7899C72.2594 19.7899 70.7586 19.1602 70.6262 17.4061C70.5821 17.0912 70.5821 16.9563 70.5821 16.7764V12.7734C70.5821 12.5935 70.5821 12.2787 70.6262 12.1437C70.8028 10.4796 72.127 9.75992 73.9368 9.75992C75.3494 9.75992 77.0709 10.1647 77.2475 12.1437C77.2916 12.4136 77.2916 12.6385 77.2475 12.7284V13.0883H74.8197V12.5935C74.8197 12.5935 74.8197 12.3686 74.7755 12.2337C74.7314 12.0538 74.5548 11.559 73.8927 11.559C73.2306 11.559 73.054 12.0088 73.0098 12.2337C72.9657 12.3236 72.9657 12.5035 72.9657 12.6835V17.0463C72.9657 17.1812 72.9657 17.3161 72.9657 17.4061C72.9657 17.496 73.0981 18.0808 73.8927 18.0808C74.6872 18.0808 74.8197 17.496 74.8197 17.4061C74.8197 17.2712 74.8638 17.1362 74.8638 17.0463V15.6969H73.8927V14.2577H77.2475V16.8214C77.2475 17.0013 77.2475 17.1362 77.2033 17.4511C77.1592 17.9008 77.0267 18.3056 76.806 18.6205C84.6632 14.8424 89.1657 10.7044 88.5919 7.15122ZM25.2045 19.655L23.9685 11.1542H23.9244L22.6884 19.655H20.084L21.8056 10.0748H26.0432L27.7647 19.655H25.2045ZM37.6083 19.655L37.5641 11.3341H37.52L36.0192 19.655H33.5914L32.0906 11.3341H32.0464L32.0023 19.655H29.5745L29.7952 10.0748H33.6797L34.8274 17.1812H34.8715L36.0192 10.0748H39.9036L40.1243 19.655H37.6083ZM48.9527 17.7659C48.6878 19.61 46.9222 19.9248 45.642 19.9248C43.5674 19.9248 42.2431 19.0253 42.2431 17.1362V16.3716H44.8034V17.3161C44.8034 17.8109 45.1565 18.1257 45.6862 18.1257C46.1717 18.1257 46.4807 17.9458 46.569 17.496C46.6132 17.3161 46.6132 17.0463 46.569 16.8214C46.0835 15.607 42.817 15.0223 42.4197 13.0883C42.3314 12.6385 42.3314 12.2787 42.3756 11.8289C42.6404 10.0748 44.3178 9.71494 45.642 9.71494C46.8339 9.71494 47.7167 9.98481 48.2023 10.5245C48.5995 10.9293 48.7761 11.4691 48.7761 12.1437V12.8184H46.3925V12.1887C46.3925 11.649 46.0835 11.3791 45.5538 11.3791C45.1123 11.3791 44.8034 11.604 44.7151 12.0088C44.7151 12.0987 44.6709 12.2787 44.7151 12.5035C44.9799 13.583 48.5113 14.2127 48.8644 16.1917C48.9968 16.4616 49.041 17.0013 48.9527 17.7659ZM57.7369 16.9113C57.7369 17.0912 57.7369 17.4511 57.6927 17.541C57.5603 19.1152 56.4567 19.9248 54.4262 19.9248C52.3957 19.9248 51.2922 19.1152 51.1156 17.541C51.1156 17.4511 51.0715 17.0912 51.0715 16.9113V10.0298H53.4993V17.0912C53.4993 17.2712 53.4993 17.3611 53.4993 17.4511C53.5434 17.631 53.6758 18.1257 54.3821 18.1257C55.0442 18.1257 55.2208 17.631 55.2649 17.4511C55.2649 17.3611 55.2649 17.2262 55.2649 17.0912V10.0298H57.6927C57.7369 10.0298 57.7369 16.9113 57.7369 16.9113ZM68.1984 19.52H64.7995L62.5483 11.9188H62.5042L62.6366 19.52H60.2971V10.0298H63.8284L65.9472 17.3161H65.9913L65.8589 10.0298H68.2426V19.52H68.1984Z"
                    fill="#3C3C3C"
                  />
                </svg>

                <h2>Samsung</h2>
              </div>

              <p>
                Publish your PWA to the Samsung Galaxy Store to make your app more discoverable to users with Samsung Galaxy Android devices.
              </p>

              <section class="platformDownloadBar">
                <Download
                  class="platformDownloadButton"
                  platform="samsung"
                  message="Download"
                  aria-label="Download Samsung Package"
                  v-on:downloadPackageError="showPackageDownloadError($event)"
                />
              </section>
            </div>

            <div
              id="pwaWindowsCard"
              class="pwaCard"
              @mouseover="platCardHover($event)"
              @mouseleave="platCardUnHover($event)"
            >
              <div class="pwaCardHeaderBlock" id="windowsCardHeaderBlock">
                <div id="windowsCardHeader">
                  <i class="fab fa-windows platformIcon" aria-label="Windows Icon"></i>
                  <h2>Windows</h2>
                </div>
              </div>

              <p>
                Publish your PWA to the Microsoft Store to make it available to the 1 billion Windows and XBox users worldwide.
              </p>

              <section class="platformDownloadBar">
                <button
                  class="platformDownloadButton"
                  @click="openWindowsModal()"
                  aria-label="Open Windows Modal"
                >
                  <i class="fas fa-long-arrow-alt-down" aria-label="Open Windows Icon"></i>
                </button>
              </section>
            </div>

            <div
              @mouseover="platCardHover($event)"
              @mouseleave="platCardUnHover($event)"
              id="pwaMacosCard"
              class="pwaCard"
            >
              <div class="pwaCardHeaderBlock">
                <i class="fab fa-apple platformIcon" aria-label="Apple Icon"></i>
                <h2>MacOS</h2>
              </div>

              <p>
                Publish your app to the MacOS Store to make your PWA available to MacOS users. Your download will contain an Xcode project which you can build and submit to the MacOS Store.
              </p>

              <section class="platformDownloadBar">
                <Download
                  class="platformDownloadButton"
                  platform="macos"
                  message="Download"
                  aria-label="Download"
                  v-on:downloadPackageError="showPackageDownloadError($event)"
                />
              </section>
            </div>
          </ul>
        </div>
        <div id="skeletonSpan" v-if="!manifest">
          <ul>
            <li>
              <span class="skeletonSpan"></span>
            </li>
            <li>
              <span class="skeletonSpan"></span>
            </li>
            <li>
              <span class="skeletonSpan"></span>
            </li>
            <li>
              <span class="skeletonSpan"></span>
            </li>
          </ul>
        </div>
      </section>
    </section>

    <section id="bottomSection">
      <div id="coolPWAs">
        <h2>Scope out rad PWAs</h2>

        <p>
          Pinterest, Spotify, and more built some PWAs and they are like whoa!
          Check them out by clicking on the image or logos. Love doing PWAs?
          Submit your own!
        </p>

        <div id="iconGrid">
          <div>
            <i class="fab fa-pinterest platformIcon"></i>
          </div>
          <div>
            <i class="fab fa-spotify platformIcon"></i>
          </div>
          <div>
            <i class="fab fa-microsoft platformIcon"></i>
          </div>
        </div>
      </div>

      <div id="bottomImageSection">
        <span>I will hold things</span>
      </div>
    </section>

    <footer>
      <p>
        PWA Builder was founded by Microsoft as a community guided, open source project to help move PWA adoption forward.
        <a
          href="https://privacy.microsoft.com/en-us/privacystatement"
        >Our Privacy Statement</a>

        <a class="termsOfUse" href="https://github.com/pwa-builder/PWABuilder/blob/master/TERMS_OF_USE.md">Terms of Use</a>
      </p>
    </footer>
  </main>
</template>

<script lang="ts">
import Vue from "vue";
import Component from "nuxt-class-component";
import { Action, State, namespace } from "vuex-class";
import GeneratorMenu from "~/components/GeneratorMenu.vue";
import StartOver from "~/components/StartOver.vue";
import Download from "~/components/Download.vue";
import Publish from "~/components/Publish.vue";
import Loading from "~/components/Loading.vue";
import Modal from "~/components/Modal.vue";
import PublishCard from "~/components/PublishCard.vue";
import Toolbar from "~/components/Toolbar.vue";
import HubHeader from "~/components/HubHeader.vue";
import {FormWizard, TabContent} from 'vue-form-wizard';
import 'vue-form-wizard/dist/vue-form-wizard.min.css';
import {VueTabs, VTab} from 'vue-nav-tabs';
import 'vue-nav-tabs/themes/vue-tabs.css';
import * as publish from "~/store/modules/publish";
import { findSuitableIcon } from "~/utils/icon-utils";
import {
  name as generatorName,
  Icon,
  Manifest,
} from "~/store/modules/generator";
import {
  validateAndroidOptions,
  generatePackageId,
} from "~/utils/android-utils";

import { validateWindowsOptions, generateWindowsPackageId } from "~/utils/windows-utils";

const PublishState = namespace(publish.name, State);
const PublishAction = namespace(publish.name, Action);
const AppGalleryPublishState = namespace(publish.name, State);
const AppGalleryPublishAction = namespace(publish.name, Action);
const GeneratorState = namespace(generatorName, State);
const GeneratorAction = namespace(generatorName, Action);

@Component({
  components: {
    GeneratorMenu,
    Loading,
    Download,
    Publish,
    StartOver,
    Modal,
    PublishCard,
    Toolbar,
    HubHeader,
    FormWizard,
    TabContent,
    VueTabs,
    VTab,
  },
})
export default class extends Vue {
  public appxForm: publish.AppxParams = {
    publisher: null,
    publisher_id: null,
    package: null,
    version: null,
  };

  // Set default web checked items
  public files: any[] = [
    "manifest",
    "serviceWorkers",
    "apiSamples",
    "windows10Package",
  ];

  @GeneratorState manifest: Manifest;
  @GeneratorState manifestUrl: string;
  @GeneratorState icons: Icon[];
  @GeneratorAction uploadIcon;
  @GeneratorAction generateMissingImages;
  @GeneratorAction updateManifest;
  @GeneratorAction updateLinkFromStorage;
  @GeneratorAction getManifestInformation;

  @PublishState appXLink: string;
  @PublishState downloadDisabled: boolean;

  @PublishAction updateStatus;
  @PublishAction disableDownloadButton;
  @PublishAction enableDownloadButton;

  public appxError: string | null = null;
  public androidPWAErrors: string[] = [];
  public appGalleryPWAError: string | null = null;
  public appGalleryPWAWarning: string | null = null;
  public apkDownloaded: boolean = false;
  public modalStatus = false;
  public openAndroid: boolean = false;
  public openAppGallery: boolean = false;
  public openWindows: boolean = false;
  public openTeams: boolean = false;
  public showBackground: boolean = false;

  public samsungDevice: boolean = false;
  public androidDevice: boolean = false;
  public iphoneDevice: boolean = false;
  public pcDevice: boolean = true;
  public macDevice: boolean = false;
  public teamsDevice: boolean = false;

  public uploadColorLoaderActive: boolean = false;
  public uploadOutlineLoaderActive: boolean = false;
  public androidForm: publish.AndroidApkOptions | null = null;
  public appGalleryForm: publish.AppGalleryApkOptions | null = null;
  public appGalleryPublish: publish.AppGalleryPublish | null = null;
  public androidFormCopyForCancellation: publish.AndroidApkOptions | null = null;
  public teamsForm: publish.TeamsParams | null = null;

  public windowsForm: publish.WindowsPackageOptions | null = null;
  public windowsFormCopyForCancellation: publish.WindowsPackageOptions | null = null;
  private windowsAnaheimForm: publish.WindowsPackageOptions | null = null;
  private windowsSpartanForm: publish.WindowsPackageOptions | null = null;
  public windowsOptionsErrors: string[] = [];
  public windowsOptionsApplied: boolean = false;
  public getIconStatus = false;
  public getIconFailed = false;
  public iconsArray = [];
  public getIconInput = "";

  private readonly maxKeyFileSizeInBytes = 2097152; // 2MB. Typically, Android keystore files are ~3KB.

  private webadb: any;
  private connected: boolean = false;
  private apk: Blob;
  private packageErrorMessage: string | null = null;
  private appGalleryPublishSuccessMessage: string | null = null;
  private reportPackageErrorUrl: string | null;
  installing: boolean = false;

  public created(): void {
    this.updateStatus();
    if(this.manifest){
      this.androidForm = this.createAndroidPackageOptionsFromManifest();
      this.windowsAnaheimForm = this.createWindowsPackageOptionsFromManifest("anaheim");
      this.windowsSpartanForm = this.createWindowsPackageOptionsFromManifest("spartan");
      this.windowsForm = this.windowsSpartanForm;
      this.appGalleryForm = this.createAppGalleryParamsFromManifest();
      this.appGalleryPublish = this.createAppGalleryPublish();
    }
  }

  showInstall(event) {
    if ((navigator as any).usb) {
      this.apkDownloaded = true;
      this.apk = event.detail;
    }
  }

  showPackageDownloadError(e: any) {
    this.packageErrorMessage = e.detail;
    this.reportPackageErrorUrl = this.getReportErrorUrl(e.detail, e.platform);
    console.error(this.packageErrorMessage, this.reportPackageErrorUrl);
  }

  showAppGalleryPublishSuccess(e: any) {
    this.appGalleryPublishSuccessMessage = e.detail;
    console.log(this.appGalleryPublishSuccessMessage);
  }

  getReportErrorUrl(errorMessage: string, platform: string): string {
    if (!errorMessage) {
      return "https://github.com/pwa-builder/pwabuilder/issues/new";
    }

    const quotedError = ">" + errorMessage.split("\n").join("\n>"); // Append the Github markdown for quotes to each line.
    const title = encodeURIComponent(`Error generating ${platform} package`);
    const message = encodeURIComponent(
      `I received the following error when generating a package for ${
        this.manifest.url || "my app"
      }\n\n${quotedError}`
    );
    return `https://github.com/pwa-builder/pwabuilder/issues/new?title=${title}&body=${message}`;
  }

  async connectDevice() {
    try {
      let webusb = await (window as any).Adb.open("WebUSB");
      let adb = await webusb.connectAdb("host::");

      this.webadb = adb;

      console.log(this.webadb);

      this.connected = true;
      console.log(this.connected);
    } catch (err) {
      console.error(err);

      this.androidPWAErrors = [err];
    }
  }

  async installAPK() {
    if (this.webadb) {
      try {
        this.installing = true;

        let sync = await this.webadb.sync();

        await sync.push(
          this.apk,
          `/data/local/tmp/pwabuilder`,
          "0644",
          async (done, total) => {
            console.log(done);
            console.log(total);
          }
        );

        await sync.quit();

        let shell = await this.webadb.shell(
          `pm install -r /data/local/tmp/pwabuilder`
        );
        let r = await shell.receive();

        let decoder = new TextDecoder();

        while (r.cmd == "WRTE") {
          if (r.data != null) {
            // Log the data decoder.decode(r.data)
            console.log(decoder.decode(r.data));
          }

          shell.send("OKAY");
          r = await shell.receive();
        }

        this.installing = false;
      } catch (err) {
        this.androidPWAErrors = [err.toString()];
        this.installing = false;
      }
    }
  }

  createWindowsPackageOptionsFromManifest(windowsConfiguration: "anaheim" | "spartan"): publish.WindowsPackageOptions {
    const pwaUrl = this.manifest.url;
    if (!pwaUrl) {
      throw new Error("Can't find the current URL");
    }

    const name = this.manifest.short_name || this.manifest.name || "My PWA";
    const packageID = generateWindowsPackageId(new URL(pwaUrl).hostname);
    const manifestIcons = this.manifest.icons || [];
    const icon =
      findSuitableIcon(manifestIcons, "any", 512, 512, "image/png") ||
      findSuitableIcon(manifestIcons, "any", 192, 192, "image/png") ||
      findSuitableIcon(manifestIcons, "any", 512, 512, "image/jpeg") || 
      findSuitableIcon(manifestIcons, "any", 192, 192, "image/jpeg") ||
      findSuitableIcon(manifestIcons, "any", 512, 512, undefined) || // Fallback to a 512x512 with an undefined type.
      findSuitableIcon(manifestIcons, "any", 192, 192, undefined) || // Fallback to a 192x192 with an undefined type.
      findSuitableIcon(manifestIcons, "any", 0, 0, "image/png") || // No large PNG and no large JPG? See if we have *any* PNG
      findSuitableIcon(manifestIcons, "any", 0, 0, "image/jpeg") || // No large PNG and no large JPG? See if we have *any* JPG
      findSuitableIcon(manifestIcons, "any", 0, 0, undefined); // Welp, we sure tried. Grab any image available.

    const packageOptions: publish.WindowsPackageOptions = {
      name: name,
      packageId: packageID,
      url: pwaUrl,
      version: windowsConfiguration === "spartan" ? "1.0.0" : "1.0.1",
      allowSigning: true,
      publisher: {
        displayName: "Contoso, Inc.",
        commonName: "CN=3a54a224-05dd-42aa-85bd-3f3c1478fdca"
      },
      generateModernPackage: windowsConfiguration === "anaheim",
      classicPackage: {
        generate: windowsConfiguration === "anaheim",
        version: "1.0.0",
        url: pwaUrl,
      },
      edgeHtmlPackage: {
        generate: windowsConfiguration === "spartan"
      },
      manifestUrl: this.manifestUrl,
      manifest: this.manifest,
      images: {
        baseImage: icon && icon.src ? icon.src : "",
        backgroundColor: "transparent",
        padding: 0.3
      }
    };
    
    return packageOptions;
  }

  createAndroidPackageOptionsFromManifest(): publish.AndroidApkOptions {
    const pwaUrl = this.manifest.url;
    if (!pwaUrl) {
      throw new Error("Can't find the current URL");
    }

    const appName = this.manifest.short_name || this.manifest.name || "My PWA";
    const packageName = generatePackageId(new URL(pwaUrl).hostname);

    // Use standalone display mode unless the manifest has fullscreen specified.
    const display =
      this.manifest.display === "fullscreen" ? "fullscreen" : "standalone";

    // StartUrl must be relative to the host.
    // We make sure it is below.
    let relativeStartUrl: string;
    if (!this.manifest.start_url || this.manifest.start_url === "/" || this.manifest.start_url === "." || this.manifest.start_url === "./") {
      // First, if we don't have a start_url in the manifest, or it's just "/",
      // then we can just use that.
      relativeStartUrl = "/";
    } else {
      // The start_url in the manifest is either a relative or absolute path.
      // Ensure it's a path relative to the root.
      const absoluteStartUrl = new URL(this.manifest.start_url, pwaUrl);
      relativeStartUrl =
        absoluteStartUrl.pathname + (absoluteStartUrl.search || "");
    }

    const manifestIcons = this.manifest.icons || [];
    const icon =
      findSuitableIcon(manifestIcons, "any", 512, 512, "image/png") ||
      findSuitableIcon(manifestIcons, "any", 192, 192, "image/png") ||
      findSuitableIcon(manifestIcons, "any", 512, 512, "image/jpeg") ||
      findSuitableIcon(manifestIcons, "any", 192, 192, "image/jpeg") ||
      findSuitableIcon(manifestIcons, "any", 512, 512, undefined) || // A 512x512 or larger image with unspecified type
      findSuitableIcon(manifestIcons, "any", 192, 192, undefined) || // A 512x512 or larger image with unspecified type
      findSuitableIcon(manifestIcons, "any", 0, 0, undefined); // Welp, we tried. Any image of any size, any type.
    const maskableIcon =
      findSuitableIcon(manifestIcons, "maskable", 512, 512, "image/png") ||
      findSuitableIcon(manifestIcons, "maskable", 192, 192, "image/png") ||
      findSuitableIcon(manifestIcons, "maskable", 192, 192, undefined);
    const monochromeIcon =
      findSuitableIcon(manifestIcons, "monochrome", 512, 512, "image/png") ||
      findSuitableIcon(manifestIcons, "monochrome", 192, 192, "image/png") || 
      findSuitableIcon(manifestIcons, "monochrome", 192, 192, undefined);
    const navColorOrFallback = 
      this.manifest.theme_color ||
        this.manifest.background_color ||
        "#000000";
    return {
      appVersion: "1.0.0.0",
      appVersionCode: 1,
      backgroundColor:
        this.manifest.background_color ||
        this.manifest.theme_color ||
        "#FFFFFF",
      display: display,
      enableNotifications: true,
      enableSiteSettingsShortcut: true,
      fallbackType: "customtabs",
      features: {
        locationDelegation: {
          enabled: true
        },
        playBilling: {
          enabled: false
        }
      },
      host: pwaUrl,
      iconUrl: icon ? icon.src : "",
      includeSourceCode: false,
      isChromeOSOnly: false,
      launcherName: this.manifest.short_name || appName, // launcher name should be the short name. If none is available, fallback to the full app name.
      maskableIconUrl: maskableIcon ? maskableIcon.src : "",
      monochromeIconUrl: monochromeIcon ? monochromeIcon.src : "",
      name: appName,
      navigationColor: navColorOrFallback,
      navigationColorDark: navColorOrFallback,
      navigationDividerColor: navColorOrFallback,
      navigationDividerColorDark: navColorOrFallback,
      orientation: this.manifest.orientation || "default",
      packageId: packageName,
      shortcuts: this.manifest.shortcuts || [],
      signing: {
        file: null,
        alias: "my-key-alias",
        fullName: `${
          this.manifest.short_name || this.manifest.name || "App"
        } Admin`,
        organization: this.manifest.name || "PWABuilder",
        organizationalUnit: "Engineering",
        countryCode: "US",
        keyPassword: "", // If empty, one will be generated by CloudAPK service
        storePassword: "", // If empty, one will be generated by CloudAPK service
      },
      signingMode: "new",
      splashScreenFadeOutDuration: 300,
      startUrl: relativeStartUrl,
      themeColor: this.manifest.theme_color || "#FFFFFF",
      shareTarget: this.manifest.share_target,
      webManifestUrl: this.manifestUrl,
    };
  }

  createAppGalleryParamsFromManifest(): publish.AppGalleryApkOptions {
    const pwaUrl = this.manifest.url;
    if (!pwaUrl) {
      throw new Error("Can't find the current URL");
    }

    const appName = this.manifest.short_name || this.manifest.name || this.manifest.json.short_name || this.manifest.json.name || "My PWA";
    const packageName = generatePackageId(new URL(pwaUrl).hostname);

    // Use standalone display mode unless the manifest has fullscreen specified.
    const display =
      this.manifest.display === "fullscreen" ? "fullscreen" : "standalone";

    // StartUrl must be relative to the host.
    // We make sure it is below.
    let relativeStartUrl: string;
    if (!this.manifest.start_url || this.manifest.start_url === "/" || this.manifest.start_url === "." || this.manifest.start_url === "./") {
      // First, if we don't have a start_url in the manifest, or it's just "/",
      // then we can just use that.
      relativeStartUrl = "/";
    } else {
      // The start_url in the manifest is either a relative or absolute path.
      // Ensure it's a path relative to the root.
      const absoluteStartUrl = new URL(this.manifest.start_url, pwaUrl);
      relativeStartUrl =
        absoluteStartUrl.pathname + (absoluteStartUrl.search || "");
    }

    const manifestIcons = this.manifest.icons || [];
    const icon =
      findSuitableIcon(manifestIcons, "any", 512, 512, "image/png") ||
      findSuitableIcon(manifestIcons, "any", 192, 192, "image/png") ||
      findSuitableIcon(manifestIcons, "any", 512, 512, "image/jpeg") ||
      findSuitableIcon(manifestIcons, "any", 192, 192, "image/jpeg") ||
      findSuitableIcon(manifestIcons, "any", 512, 512, undefined) || // A 512x512 or larger image with unspecified type
      findSuitableIcon(manifestIcons, "any", 192, 192, undefined) || // A 512x512 or larger image with unspecified type
      findSuitableIcon(manifestIcons, "any", 0, 0, undefined);  // Welp, we tried. Any image of any size, any type.
      // this.manifest.icon[0];
    const maskableIcon =
      findSuitableIcon(manifestIcons, "maskable", 512, 512, "image/png") ||
      findSuitableIcon(manifestIcons, "maskable", 192, 192, "image/png") ||
      findSuitableIcon(manifestIcons, "maskable", 192, 192, undefined);
    const monochromeIcon =
      findSuitableIcon(manifestIcons, "monochrome", 512, 512, "image/png") ||
      findSuitableIcon(manifestIcons, "monochrome", 192, 192, "image/png") || 
      findSuitableIcon(manifestIcons, "monochrome", 192, 192, undefined);

    return {
      packageId: packageName,
      name: appName,
      launcherName: this.manifest.short_name || this.manifest.json.short_name || appName, // launcher name should be the short name. If none is available, fallback to the full app name.
      appVersion: "1.0.0.0",
      appVersionCode: 1,
      display: display,
      host: pwaUrl,
      startUrl: relativeStartUrl,
      webManifestUrl: this.manifestUrl || "",
      themeColor: this.manifest.theme_color || "#FFFFFF",
      navigationColor:
        this.manifest.theme_color || this.manifest.background_color || "#000000",
      backgroundColor:
        this.manifest.background_color ||
        this.manifest.theme_color ||
        "#FFFFFF",
      iconUrl: icon ? icon.src : "",
      maskableIconUrl: maskableIcon ? maskableIcon.src : "",
      monochromeIconUrl: monochromeIcon ? monochromeIcon.src : "",
      signingMode: "new",
      signing: {
        file: null,
        alias: "my-key-alias",
        fullName: `${this.manifest.short_name.replace(/[^\w\s]/gi, '') || this.manifest.name.replace(/[^\w\s]/gi, '') || "App"} Admin`,
        organization: this.manifest.name.replace(/[^\w\s]/gi, '') || "PWABuilder",
        organizationalUnit: "Engineering",
        countryCode: "US",
        keyPassword: "", // If empty, one will be generated by CloudAPK service
        storePassword: "" // If empty, one will be generated by CloudAPK service
      },
      shortcuts: this.manifest.shortcuts || [],
      fallbackType: "customtabs",
      splashScreenFadeOutDuration: 300,
      enableNotifications: false,
      whitelist: this.whitelist,
    };
  }

  createAppGalleryPublish(): publish.AppGalleryPublish {
    return {
      client_id: "",
      client_key: "",
      app_id: "",
      apk: ""
    };
  }

  public mounted(): void {
    const overrideValues = {
      uri: window.location.href,
      pageName: "publishPage",
      pageHeight: window.innerHeight,
    };

    if (this.manifest && this.manifest.description && this.teamsForm) {
      if (this.manifest.description.length > 80) {
        this.teamsForm.longDescription = this.manifest.description;
      } else {
        this.teamsForm.shortDescription = this.manifest.description;
      }
    }

    if (!this.manifest) {
      const currentURL = sessionStorage.getItem("currentURL");
      if (currentURL) {
        try {
          this.updateLinkFromStorage(currentURL);
          this.getManifestInformation();
          this.enableDownloadButton();
        } catch (err) {
          console.error(err);
        }
      } else {
        this.disableDownloadButton();
      }
    } else {
      this.enableDownloadButton();
    }

    this.$awa(overrideValues);
  }

  /**
   * Called when the user changes the signing mode.
   */
  androidSigningModeChanged() {
    if (!this.androidForm || !this.androidForm.signing) {
      return;
    }

    // If the user chose "mine", clear out existing values.
    if (this.androidForm.signingMode === "mine") {
      this.androidForm.signing.alias = "";
      this.androidForm.signing.fullName = "";
      this.androidForm.signing.organization = "";
      this.androidForm.signing.organizationalUnit = "";
      this.androidForm.signing.countryCode = "";
      this.androidForm.signing.keyPassword = "";
      this.androidForm.signing.storePassword = "";
    } else if (this.androidForm.signingMode === "new") {
      this.androidForm.signing.alias = "my-key-alias";
      this.androidForm.signing.fullName = `${
        this.manifest.short_name || this.manifest.name || "App"
      } Admin`;
      this.androidForm.signing.organization =
        this.manifest.name || "PWABuilder";
      this.androidForm.signing.organizationalUnit = "Engineering";
      this.androidForm.signing.countryCode = "US";
      this.androidForm.signing.keyPassword = "";
      this.androidForm.signing.storePassword = "";
      this.androidForm.signing.file = null;
    }
  }

  appGallerySigningModeChanged() {
    if (!this.appGalleryForm || !this.appGalleryForm.signing) {
      return;
    }

    // If the user chose "mine", clear out existing values.
    if (this.appGalleryForm.signingMode === "mine") {
      this.appGalleryForm.signing.alias = "";
      this.appGalleryForm.signing.fullName = "";
      this.appGalleryForm.signing.organization = "";
      this.appGalleryForm.signing.organizationalUnit = "";
      this.appGalleryForm.signing.countryCode = "";
      this.appGalleryForm.signing.keyPassword = "";
      this.appGalleryForm.signing.storePassword = "";
    } else if (this.appGalleryForm.signingMode === "new") {
      this.appGalleryForm.signing.alias = "my-key-alias";
      this.appGalleryForm.signing.fullName = `${this.manifest.short_name || this.manifest.name || "App"} Admin`;
      this.appGalleryForm.signing.organization = this.manifest.name || "PWABuilder";
      this.appGalleryForm.signing.organizationalUnit = "Engineering";
      this.appGalleryForm.signing.countryCode = "US";
      this.appGalleryForm.signing.keyPassword = "";
      this.appGalleryForm.signing.storePassword = "";
      this.appGalleryForm.signing.file = null;
    }
  }

  get androidPasswordPlaceholder(): string {
    if (!this.androidForm) {
      return "";
    }

    if (this.androidForm.signingMode === "new") {
      return "Type a new password or leave empty to generate one";
    }

    return "";
  }

  get appGalleryPasswordPlaceholder(): string {
    if (!this.appGalleryForm) {
      return "";
    }

    if (this.appGalleryForm.signingMode === "new") {
      return "Type a new password or leave empty to use a generated one";
    }

    return "";
  }

  /**
   * Called when the user uploads their Android keystore signing file.
   */
  androidSigningKeyUploaded(event: InputEvent) {
    if (!this.androidForm || !this.androidForm.signing) {
      return;
    }

    const signing = this.androidForm.signing;
    const filePicker = event.target as HTMLInputElement;
    if (filePicker && filePicker.files && filePicker.files.length > 0) {
      const keyFile = filePicker.files[0];

      // Make sure it's a reasonable size.
      if (keyFile.size > this.maxKeyFileSizeInBytes) {
        console.error("Keystore file is too large.", {
          maxSize: this.maxKeyFileSizeInBytes,
          fileSize: keyFile.size,
        });
        this.androidForm.signingMode = "none";
      }

      // Read it in as a Uint8Array and store it in our signing object.
      const fileReader = new FileReader();
      fileReader.onload = () => (signing.file = fileReader.result as string);
      fileReader.onerror = (progressEvent) => {
        console.error(
          "Unable to read keystore file",
          fileReader.error,
          progressEvent
        );
        signing.file = null;
        if (this.androidForm) {
          this.androidForm.signingMode = "none";
        }
      };

      fileReader.readAsDataURL(keyFile);
    }
  }

  appGallerySigningKeyUploaded(event: InputEvent) {
    if (!this.appGalleryForm || !this.appGalleryForm.signing) {
      return;
    }

    const signing = this.appGalleryForm.signing;
    const filePicker = event.target as HTMLInputElement;
    if (filePicker && filePicker.files && filePicker.files.length > 0) {
      const keyFile = filePicker.files[0];

      // Make sure it's a reasonable size.
      if (keyFile.size > this.maxKeyFileSizeInBytes) {
        console.error("Keystore file is too large.", {
          maxSize: this.maxKeyFileSizeInBytes,
          fileSize: keyFile.size
        });
        this.appGalleryForm.signingMode = "none";
      }

      // Read it in as a Uint8Array and store it in our signing object.
      const fileReader = new FileReader();
      fileReader.onload = () => (signing.file = fileReader.result as string);
      fileReader.onerror = progressEvent => {
        console.error(
          "Unable to read keystore file",
          fileReader.error,
          progressEvent
        );
        signing.file = null;
        if (this.appGalleryForm) {
          this.appGalleryForm.signingMode = "none";
        }
      };

      fileReader.readAsDataURL(keyFile);
    }
  }

  platCardHover(ev) {
    if (ev.target) {
      const parent = ev.target.children[2];

      let targetButton: HTMLButtonElement | null = null;

      if (parent) {
        targetButton = ev.target.children[2].children[0];
      }

      if (targetButton) {
        targetButton.classList.add("platformDownloadButtonHover");
      }
    }
  }

  platCardUnHover(ev) {
    const targetButton: HTMLButtonElement = ev.target.children[2].children[0];

    if (targetButton) {
      targetButton.classList.remove("platformDownloadButtonHover");
    }
  }

  public goToHome(): void {
    this.$router.push({
      path: this.$i18n.path(""),
    });
  }

  public openAppXModal(): void {
    this.openWindows = false;
    (this.$refs.appxModal as Modal).show();
  }

  public openAndroidOptionModal(): void {
    this.openAndroid = false;

    // Create a copy of the Android form. If the user cancels the dialog, we'll revert back to this copy.
    if (this.androidForm) {
      this.androidFormCopyForCancellation = { ...this.androidForm };
      if (this.androidForm.signing) {
        this.androidFormCopyForCancellation.signing = {
          ...this.androidForm.signing,
        };
      }
    }

    (this.$refs.androidPWAModal as Modal).show();
  }

  public openAndroidModal(): void {
    this.openAndroid = true;
  }

  public closeAndroidModal(): void {
    this.openAndroid = false;
    this.openWindows = false;
  }

  public openAppGalleryOptionModal(): void {
    this.openAppGallery = false;

    // Create a copy of the Android form. If the user cancels the dialog, we'll revert back to this copy.
    if (this.appGalleryForm) {
      this.appGalleryFormCopyForCancellation = { ...this.appGalleryForm };
      if (this.appGalleryForm.signing) {
        this.appGalleryFormCopyForCancellation.signing = {
          ...this.appGalleryForm.signing
        };
      }
    }

    (this.$refs.appGalleryPWAModal as Modal).show();
  }

  public openAppGalleryModal(): void {
    this.openAppGallery = true;
    this.checkIconAndManifest();
  }

  public closeAppGalleryModal(): void {
    this.openAppGallery = false;
    this.openWindows = false;
  }

  public openWindowsModal(): void {
    this.openWindows = true;
  }

  public openWindowsOptionsModal(config: "anaheim" | "spartan"): void {
    this.openWindows = false;
    this.windowsFormConfiguration = config;

   // Create a copy of the Windows form. If the user cancels the dialog, we'll revert back to this copy.
    if (this.windowsForm) {
      this.windowsFormCopyForCancellation = { ...this.windowsForm };
    }

    (this.$refs.windowsPWAModal as Modal).show();
    Vue.prototype.$awa({ referrerUri: 'https://www.pwabuilder.com/publish/windows10-appx' });
  }

  public openTeamsModal(): void {
    this.openTeams = true;
    this.disableDownloadButton();
  }

  public closeWindowsModal(): void {
    this.openWindows = false;
  }

  public closeTeamsModal(): void {
    this.openTeams = false;

    this.teamsForm = {
      publisherName: null,
      shortDescription: null,
      longDescription: null,
      privacyUrl: null,
      termsOfUseUrl: null,
      colorImageFile: null,
      outlineImageFile: null,
    };

    if (this.manifest && this.manifest.description) {
      if (this.manifest.description.length > 80) {
        this.teamsForm.longDescription = this.manifest.description;
      } else {
        this.teamsForm.shortDescription = this.manifest.description;
      }
    }

    this.enableDownloadButton();
  }

  public async handleUploadColorIcon(): Promise<void> {
    if (!this.teamsForm) {
      return;
    }

    this.uploadColorLoaderActive = true;
    const el = <HTMLInputElement>(
      document.getElementById("upload-file-input-color")
    );
    if (el && el.files) {
      this.teamsForm.colorImageFile = el.files[0];
    }

    await this.uploadIcon(this.teamsForm.colorImageFile);
    await this.updateManifest(this.manifest);
    this.validateTeamsForm();
    this.uploadColorLoaderActive = false;
  }

  public clickUploadColorFileInput(): void {
    const el = document.getElementById("upload-file-input-color");
    if (el) {
      el.click();
    }
  }

  public async handleUploadOutlineIcon(): Promise<void> {
    if (!this.teamsForm) {
      return;
    }

    this.uploadOutlineLoaderActive = true;
    const el = <HTMLInputElement>(
      document.getElementById("upload-file-input-outline")
    );
    if (el && el.files) {
      this.teamsForm.outlineImageFile = el.files[0];
    }

    await this.uploadIcon(this.teamsForm.outlineImageFile);
    await this.updateManifest(this.manifest);
    this.validateTeamsForm();
    this.uploadOutlineLoaderActive = false;
  }

  public clickUploadOutlineFileInput(): void {
    const el = document.getElementById("upload-file-input-outline");
    if (el) {
      el.click();
    }
  }

  public validateTeamsForm(): void {
    if (!this.teamsForm) {
      return;
    }

    const buttonDisabled = this.downloadDisabled;
    const formFilled =
      typeof this.teamsForm.publisherName === "string" &&
      typeof this.teamsForm.shortDescription === "string" &&
      typeof this.teamsForm.longDescription === "string" &&
      typeof this.teamsForm.privacyUrl === "string" &&
      typeof this.teamsForm.termsOfUseUrl === "string" &&
      this.teamsForm.colorImageFile !== null;

    if (buttonDisabled && formFilled) {
      this.enableDownloadButton();
    } else if (!buttonDisabled && !formFilled) {
      this.disableDownloadButton();
    }
  }

  public async androidOptionsModalSubmitted(): Promise<void> {
    if (!this.androidForm) {
      return;
    }

    const validationErrors = validateAndroidOptions(this.androidForm);
    if (validationErrors.length > 0) {
      this.androidPWAErrors = validationErrors.map((e) => e.error);
      return;
    }

    (this.$refs.androidPWAModal as Modal).hide();
    this.openAndroid = true;
    this.androidPWAErrors = [];
  }

  public async appGalleryOptionsModalSubmitted(): Promise<void> {
    if (!this.appGalleryForm) {
      return;
    }
    const validationErrors = validateAppGalleryOptions(this.appGalleryForm);
    if (validationErrors.length > 0) {
      this.androidPWAError = validationErrors.map(e => e.error).join(", ");
      return;
    }

    (this.$refs.appGalleryPWAModal as Modal).hide();
    this.openAppGallery = true;
    this.androidPWAError = null;
  }

  public androidOptionsModalCancelled() {
    this.androidForm =
      this.androidFormCopyForCancellation ||
      this.createAndroidPackageOptionsFromManifest();
    this.androidPWAErrors = [];
    (this.$refs.androidPWAModal as Modal).hide();
    this.openAndroid = true;
  }

  public appGalleryOptionsModalCancelled() {
    this.appGalleryForm =
      this.appGalleryFormCopyForCancellation ||
      this.createAppGalleryParamsFromManifest();
    this.androidPWAError = null;  // <--- need definition
    (this.$refs.appGalleryPWAModal as Modal).hide();  // <--- need definition
    this.openAppGallery = true;
  }

  public windowsOptionsModalCancelled() {
    this.windowsForm = 
      this.windowsFormCopyForCancellation ||
      this.createWindowsPackageOptionsFromManifest(this.windowsFormConfiguration);

    this.windowsOptionsErrors = [];

    (this.$refs.windowsPWAModal as Modal).hide();
    this.openWindows = true;
  }

  public windowsOptionsModalSubmitted() {
   if (!this.windowsForm) {
      return;
    }

    const validationErrors = validateWindowsOptions(this.windowsForm, this.windowsFormConfiguration);
    if (validationErrors.length > 0) {
      this.windowsOptionsErrors = validationErrors.map((e) => e.error);
      return;
    }

    (this.$refs.windowsPWAModal as Modal).hide();
    this.openWindows = true;
    this.windowsOptionsErrors = [];

    this.windowsOptionsApplied = true;
  }

  public ConstructErrorMessage(list) {
    if (list.length === 1) {
      return `Invalid package name. "${list[0]}" is a keyword.`;
    } else {
      return `Invalid package name. "${list
        .slice(0, list.length - 1)
        .join(", ")} and ${list[list.length - 1]}" are keywords`;
    }
  }

  public onCancelAppxModal(): void {
    this.appxForm = {
      publisher: null,
      publisher_id: null,
      package: null,
      version: null,
    };
  }

  public modalOpened() {
    window.scrollTo(0, 0);

    this.modalStatus = true;
    this.showBackground = true;
  }

  public modalClosed() {
    this.modalStatus = false;
    this.showBackground = false;
  }

  public androidModalClosed() {
    this.modalStatus = false;
    this.showBackground = false;
    this.openAndroid = true;
  }

  public appGalleryModalClosed() {
    this.modalStatus = false;
    this.showBackground = false;
    this.openAppGallery = true;
  }

  public printAppGalleryForm() {
    this.appGalleryForm.HMSKits = [];
    if(this.appGalleryForm.hmsAdsKit)
      this.appGalleryForm.HMSKits.push('ads');
    if(this.appGalleryForm.hmsAnalyticsKit)
      this.appGalleryForm.HMSKits.push('analytics');
    if(this.appGalleryForm.hmsPushKit)
      this.appGalleryForm.HMSKits.push('push');
  }

  public windowsModalClosed() {
    this.modalStatus = false;
    this.showBackground = false;
    this.openWindows = true;
  }

  get windowsFormConfiguration(): "spartan" | "anaheim" {
    if (this.windowsForm && this.windowsForm.edgeHtmlPackage && this.windowsForm.edgeHtmlPackage.generate) {
      return "spartan";
    }

    return "anaheim";
  }

  set windowsFormConfiguration(val: "spartan" | "anaheim") {
    if (val === "spartan") {
      this.windowsForm = this.windowsSpartanForm;
    } else if (val === "anaheim") {
      this.windowsForm = this.windowsAnaheimForm;
    }
  }

  public validateAPKOptions() {
    return new Promise((resolve, reject) => {
      const noKeyPassword = "Key password cannot be empty"
      const noKeyStorePassword = "Key store password cannot be empty"
      const passwordTooShort = "Key password or key store password must be at least 6 characters."
      const keyPasswordsNotMatching = "Key password and key store password must be identical."
      this.appGalleryPWAError = [];
      if(this.appGalleryForm.signing.keyPassword.length == 0) {
        console.error(noKeyPassword);
        this.appGalleryPWAError.push(noKeyPassword);
      }
      if(this.appGalleryForm.signing.storePassword.length == 0) {
        console.error(noKeyStorePassword);
        this.appGalleryPWAError.push(noKeyStorePassword);
      }
      if(this.appGalleryForm.signing.storePassword.length < 6 || this.appGalleryForm.signing.keyPassword.length < 6) {
        console.error(passwordTooShort);
        this.appGalleryPWAError.push(passwordTooShort);
      }
      if(this.appGalleryForm.signing.storePassword !== this.appGalleryForm.signing.keyPassword) {
        console.error(keyPasswordsNotMatching);
        this.appGalleryPWAError.push(keyPasswordsNotMatching);
      }
      if(this.appGalleryPWAError.length > 0) {
        console.error(this.appGalleryPWAError);
        reject(false);
      } else {
        this.appGalleryPWAError = null;
        console.log("validateAPKOptions OK")
        resolve(true);        
      }
    })
  }

  public checkIconAndManifest() {
    this.appGalleryPWAWarning = [];
    const missingIconUrl = "Warning: Icon URL is missing, please enter a valid URL for the website. If you do not specify any Icon URL, Android default icon will be used by APK builder."
    const missingWebManifest = "Warning: Manifest URL is missing, this indicates the website is not optimized for PWA. Please make sure the website is PWA compatible before proceeding. You can continue building this APK but it may not behave or perform normally."
    if(this.appGalleryForm.iconUrl.length === 0){
      console.error(missingIconUrl);
      this.appGalleryPWAWarning.push(missingIconUrl);
    }
    if(this.appGalleryForm.webManifestUrl.length === 0) {
      console.error(missingWebManifest);
      this.appGalleryPWAWarning.push(missingWebManifest);
    }
    if(this.appGalleryPWAWarning.length === 0) {
      this.appGalleryPWAWarning = null;
    }
  }

  public validateAGCHMS() {
    return new Promise((resolve, reject) => {
      console.log('validateAGCHMS now')
      this.appGalleryPWAError = [];
      const wrongFormat = "HMS Ad Is must be in 14 characters string format, please check again.";
      const missingId = "HMS Ad Is missing, pleaese select at least one type of ad and fill in the correct ads id."
      if(this.appGalleryPWAError.length > 0) {
        console.error(this.appGalleryPWAError);
        reject(false);
      } else {
        this.appGalleryForm.ads_id = [];
        if(this.appGalleryForm.hmsAdsSplash && this.appGalleryForm.hmsAdsSplashId){
          this.appGalleryForm.ads_id.push({'splash':this.appGalleryForm.hmsAdsSplashId});
        }
        if(this.appGalleryForm.hmsAdsTopBanner && this.appGalleryForm.hmsAdsTopBannerId){
          this.appGalleryForm.ads_id.push({'topBanner':this.appGalleryForm.hmsAdsTopBannerId});
        }
        if(this.appGalleryForm.hmsAdsBottomBanner && this.appGalleryForm.hmsAdsBottomBannerId){
          this.appGalleryForm.ads_id.push({'bottomBanner':this.appGalleryForm.hmsAdsBottomBannerId});
        }
        this.appGalleryPWAError = null;
        resolve(true);        
      }
    });
  }

  public checkHMSAds(type) {
    if(this.appGalleryForm.hmsAdsSplashId && type == 1){
      if(this.appGalleryForm.hmsAdsSplashId.length > 0)
        this.appGalleryForm.hmsAdsSplash = true;
      if(this.appGalleryForm.hmsAdsSplashId.length == 0){
        this.appGalleryForm.hmsAdsSplash = false;
      }
    }
    if(this.appGalleryForm.hmsAdsTopBannerId && type == 2){
      if(this.appGalleryForm.hmsAdsTopBannerId.length > 0)
        this.appGalleryForm.hmsAdsTopBanner = true;
      if(this.appGalleryForm.hmsAdsTopBannerId.length == 0){
        this.appGalleryForm.hmsAdsTopBanner = false;
      }
    }
    if(this.appGalleryForm.hmsAdsBottomBannerId && type == 3){
      if(this.appGalleryForm.hmsAdsBottomBannerId.length > 0)
        this.appGalleryForm.hmsAdsBottomBanner = true;
      if(this.appGalleryForm.hmsAdsBottomBannerId.length == 0){
        this.appGalleryForm.hmsAdsBottomBanner = false;
      }
    }
  }

  public onAGCSFileChange(e){
    let files = e.target.files || e.dataTransfer.files;
    if (!files.length) return;
    this.readAGCSFile(files[0]).then(
      data => {
        this.appGalleryForm.aGConnectServicesJSON = data;
        console.log(JSON.stringify(this.appGalleryForm.aGConnectServicesJSON));
        this.validateAGCJson();
      }
    ).then(
      this.getBase64(files[0]).then(
        data => {
          this.appGalleryForm.agcs = data;
        }
      )
    ).catch(err => console.error(err));
  }

  public validateAGCJson() {
    this.appGalleryPWAError = [];

    if(!this.appGalleryForm.aGConnectServicesJSON.client){
      this.appGalleryPWAError.push("Missing 'client' key.");
    } else {
      if(!this.appGalleryForm.aGConnectServicesJSON.client.cp_id){
        this.appGalleryPWAError.push("Missing 'client.cp_id' key.");
      }
      if(!this.appGalleryForm.aGConnectServicesJSON.client.product_id){
        this.appGalleryPWAError.push("Missing 'client.product_id' key.");
      }
      if(!this.appGalleryForm.aGConnectServicesJSON.client.client_id){
        this.appGalleryPWAError.push("Missing 'client.client_id' key.");
      }
      if(!this.appGalleryForm.aGConnectServicesJSON.client.client_secret){
        this.appGalleryPWAError.push("Missing 'client.client_secret' key.");
      }
      if(!this.appGalleryForm.aGConnectServicesJSON.client.app_id){
        this.appGalleryPWAError.push("Missing 'client.app_id' key.");
      }
      if(!this.appGalleryForm.aGConnectServicesJSON.client.package_name){
        this.appGalleryPWAError.push("Missing 'client.package_name' key.");
      } else {
        if(this.appGalleryForm.aGConnectServicesJSON.client.package_name !== this.appGalleryForm.packageId) {
          this.appGalleryPWAError.push(`The package name: "${this.appGalleryForm.aGConnectServicesJSON.client.package_name}" in the JSON is different than PWA package name: "${this.appGalleryForm.packageId}".`);
        }
      }
      if(!this.appGalleryForm.aGConnectServicesJSON.client.api_key){
        this.appGalleryPWAError.push("Missing 'client.api_key' key.");
      }
    }
    if(this.appGalleryPWAError.length > 0) {
      this.appGalleryPWAError.push(`This is not a valid agconnect-services.json, please make sure you use the correct agconnect-services.json provided by AppGallery.`)
    } else {
      this.appGalleryPWAError.length = null;
    }
  }

  public readAGCSFile(file) {
    return new Promise((resolve, reject) => {
      let reader = new FileReader();
      reader.readAsText(file);
      reader.onload = e => resolve(JSON.parse(e.target.result));
      reader.onerror = error => reject(error);
    })
  }

  public getBase64(file) {
    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      reader.readAsDataURL(file);
      reader.onload = () => resolve(reader.result);
      reader.onerror = error => reject(error);
    });
  }

  public getAGAPK(e){
    let file = e.target.files || e.dataTransfer.files;
    if (!file.length) return;
    this.getBase64(file[0]).then(
      data => {
        console.log(data);
        this.appGalleryPublish.apk = data;
      }
    ).catch(err => console.error(err));
  }

  public async getIcon(name){
    this.iconsArray = [];
    this.getIconFailed = false;
    this.getIconStatus = true;
    const getIcons = `${process.env.appGalleryPackageGeneratorUrl}/get_icons`,
          options = { name: name.replace(/ /g,"_") };
    try {
      const response = await fetch(getIcons, {
        method: "POST",
        headers: new Headers({'content-type': 'application/json'}),
        body: JSON.stringify(options)
      });

      if (response.status === 200) {
        this.getIconStatus = false;
        const data = await response.json();
        console.log(JSON.stringify(data));
        this.iconsArray = data.icons;
      } else {
        this.getIconStatus = false;
        this.getIconFailed = true;
        const responseText = await response.text();
        console.error(responseText);
      }
    } catch (err) {
      this.getIconStatus = false;
      this.getIconFailed = true;
      console.error(err);
    } finally {

    }
  }

  public changeIconUrl(url){
    this.appGalleryForm.iconUrl = url;
  }

  public clearIconUrl(){
    this.appGalleryForm.iconUrl = ""; 
  }

  public clearError(){
    if(!this.appGalleryForm.hmsAdsSplash)
      this.appGalleryForm.hmsAdsSplash = true;
    if(!this.appGalleryForm.hmsAdsTopBanner)
      this.appGalleryForm.hmsAdsTopBanner = true;
    if(!this.appGalleryForm.hmsAdsBottomBanner)
      this.appGalleryForm.hmsAdsBottomBanner = true;
    if(!this.appGalleryForm.hmsAdsSplashId)
      this.appGalleryForm.hmsAdsSplashId = "testq6zq98hecj";
    if(!this.appGalleryForm.hmsAdsTopBannerId)
      this.appGalleryForm.hmsAdsTopBannerId = "testw6vs28auh3";
    if(!this.appGalleryForm.hmsAdsBottomBannerId)
      this.appGalleryForm.hmsAdsBottomBannerId = "testw6vs28auh3";
  }
}



Vue.prototype.$awa = function (config) {
  if (awa) {
    awa.ct.capturePageView(config);
  }

  return;
};

declare var awa: any;
</script>

<style lang="scss">
/* stylelint-disable */

@import "~assets/scss/base/variables";

.termsOfUse {
  margin-left: 6px;
}

// Hide macOS card for now
#pwaMacosCard {
  display: none;
}

.pwaCardHeaderBlock #windowsCardHeader {
  display: flex;
  align-items: center;
  justify-content: center;
}

#windowsCardHeaderBlock {
  justify-content: space-between;
}

#windowsCardHeaderBlock span {
  font-weight: bold;
}

#androidInstallWrapper {
  padding: 10px;
  border: solid 1px #3c3c3c;
  border-radius: 10px;
  width: 100%;
  display: flex;
  justify-content: flex-end;
  align-items: center;
  text-align: center;
  padding-bottom: 24px;
  flex-direction: column;
  padding-left: 1em;
  padding-right: 1em;
}

#androidInstallWrapper p {
  color: #3c3c3c;
  display: block;
  margin-bottom: 2em;
  font-style: normal;
  font-weight: normal;
  font-size: 14px;
  line-height: 21px;
}

#androidInstallActions {
  display: flex;
}

.androidInstallButton {
  background: #3c3c3c;
  color: white;
  font-size: 14px;
  border-radius: 20px;
  width: 150px;
  height: 40px;
  padding-left: 20px;
  padding-right: 20px;
  font-family: sans-serif;
  font-style: normal;
  font-weight: 600;
  margin-left: 12px;
  border: none;
}

.androidInstallButton:disabled {
  background: #8a8a8a;
}

#installSpinner {
  margin-top: -1px !important;
  height: 32px;
  display: inline-block;
  padding-top: 4px;
}

@-moz-document url-prefix() {
  #installSpinner {
    margin-top: 38px !important;
    margin-left: 10px !important;
  }
}

.flavor {
  width: 32px;
  height: 32px;
  border-radius: 40px;
  overflow: hidden;
}

.flavor > .colorbands {
  position: relative;
  top: 0%;
  left: -20%;

  width: 140%;
  height: 800%;

  background-image: linear-gradient(
    0deg,
    #1fc2c8 25%,
    #9337d8 50%,
    #9337d8 75%,
    #1fc2c8 100%
  );
  background-position: 0px 0px;
  background-repeat: repeat-y;

  animation: colorbands 100s linear infinite;
  transform: rotate(180deg);
}

@keyframes colorbands {
  to {
    background-position: 0 -1000vh;
  }
}

.icon {
  position: relative;
  color: white;
  top: -25px;
  left: 7px;

  .lds-dual-ring {
    display: inline-block;
    width: 32px;
    height: 32px;
  }

  .lds-dual-ring:after {
    content: " ";
    display: block;
    width: 16px;
    height: 16px;
    margin: 1px;
    border-radius: 50%;
    border: 5px solid #fff;
    border-color: #fff transparent #fff transparent;
    animation: lds-dual-ring 1.2s linear infinite;
  }

  @keyframes lds-dual-ring {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }
}

#skeletonSpan {
  width: 30em;
  justify-content: center;

  ul {
    flex-grow: 2;
    list-style: none;
    padding: 0;
    margin: 0;
    margin-bottom: 42px;

    li {
      font-size: 14px;
      font-weight: bold;
      padding: 0.5em;
      padding-left: 0;
      display: flex;
      flex-direction: row;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 5px;
      color: #3c3c3c span {
        font-style: normal;
        font-weight: normal;
        font-size: 14px;
        line-height: 18px;
        color: #3c3c3c;
      }
    }
  }

  .skeletonSpan {
    background: linear-gradient(
      to right,
      rgba(140, 140, 140, 0.8),
      rgba(140, 140, 140, 0.18),
      rgba(140, 140, 140, 0.33)
    );
    background-size: 800px 104px;
    animation-duration: 1s;
    animation-fill-mode: forwards;
    animation-iteration-count: infinite;
    animation-name: shimmer;
    animation-timing-function: linear;
    height: 1em;
    width: 100%;
  }

  @keyframes shimmer {
    0% {
      background-position: -468px 0;
    }

    100% {
      background-position: 468px 0;
    }
  }
}

#devicePreviews {
  height: 504px;
}

#desktopDevicePreview {
  display: flex;
  justify-content: center;
  align-items: center;

  svg {
    width: 24em;
  }
}

#mobileDevicePreview {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 28em;

  svg {
    width: 18em;
    height: 53em;
  }

  img {
    height: 20em;
  }
}

@keyframes publishfadein {
  from {
    opacity: 0;
    transform: translateY(-50px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

footer {
  display: flex;
  justify-content: center;
  padding-left: 10px;
  padding-right: 10px;
  font-size: 12px;
  color: rgba(60, 60, 60, 0.5);
  background: white;
}

footer p {
  text-align: center;
  font-style: normal;
  font-weight: normal;
  font-size: 12px;
  line-height: 18px;
  color: #707070;
}

footer a {
  color: #707070;
  text-decoration: underline;
}

.pwaIcon {
  height: 22px;
  margin-right: 10px;
}

#teamsIconImg {
  height: 40px;
  width: 42px;
}

@media (max-height: 700px) {
  #scorepublishSideBySide header {
    top: 51px;
  }
}

#modalBackground {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 98999;
  will-change: opacity;
}

#appxModal .modal {
  @media (max-width: 640px) {
    .modal-box {
      width: 100%;

      .closeButtonDiv {
        margin-top: 20px;
      }
    }
  }
}

@keyframes opened {
  from {
    opacity: 0;
  }

  to {
    opacity: 0.7;
  }
}

#appxModalBody {
  input {
    padding: initial;
  }

  div {
    margin-top: 40px;

    #topLabel {
      font-weight: initial;
    }

    label {
      font-weight: bold;
      margin-bottom: 20px;
      display: block;
    }
  }
}

#publishSideBySide {
  display: flex;
  background: linear-gradient(-32deg, #9337d8, #1fc2c8);

  #publishLeftSide {
    height: 100%;
    flex: 1;

    display: flex;
    justify-content: center;
    align-self: center;
    align-items: center;

    header {
      display: flex;
      align-items: center;
      padding-left: 159px;
      margin-top: 32px;

      #headerText {
        font-size: 28px;
        font-weight: normal;
      }

      #logo {
        background: lightgrey;
        border-radius: 50%;
        width: 48px;
        height: 48px;
        margin-right: 12px;
      }
    }

    #introContainer {
      color: white;
      margin-left: 20px;
      margin-right: 20px;

      h2 {
        font-family: sans-serif;
        font-weight: 600;
        font-size: 24px;
      }

      p {
        font-size: 18px;
      }

      #publishActionsContainer {
        display: flex;
        width: 100%;

        button {
          border: none;
          width: 264px;
          border-radius: 20px;
          font-size: 18px;
          font-weight: bold;
          padding-top: 9px;
          padding-bottom: 11px;
        }

        #downloadAllButton {
          margin-right: 11px;
          background: $color-button-primary-purple-variant;
          color: white;
          width: 264px;
          border-radius: 20px;
          font-size: 18px;
          padding-top: 9px;
          padding-bottom: 11px;
          font-weight: bold;
          display: flex;
          justify-content: center;
        }

        #downloadAllButton:hover {
          cursor: pointer;
        }
      }
    }
  }

  #publishRightSide {
    height: 100%;
    flex: 2;
    display: flex;
    flex-direction: column;
    background: white;
    align-items: center;
    justify-content: center;
    padding-bottom: 100px;

    #platformsListContainer {
      padding-top: 20px;
      padding-right: 20px;
      padding-left: 20px;
      color: white;

      ul {
        list-style: none;
        padding: 0;
        margin: 0;

        display: grid;
        // flex-direction: column;
        // flex-wrap: wrap;

        .pwaCard {
          background: #f0f0f0;
          border-radius: 4px;
          color: #3c3c3c;
          padding-left: 24px;
          padding-right: 24px;
          padding-top: 20px;
          padding-bottom: 40px;
          margin: 10px;
          position: relative;
          transition: box-shadow 0.3s;

          .pwaCardHeaderBlock {
            display: flex;
            align-items: center;
          }

          .pwaCardHeaderBlock h2 {
            font-style: normal;
            font-weight: bold;
            font-size: 16px;
            line-height: 24px;
            color: rgba(60, 60, 60, 0.9);
          }

          .platformDownloadBar {
            position: absolute;
            bottom: 20px;
            right: 20px;
            height: 30px;
          }

          .pwaCardIconBlock {
            display: flex;
            align-items: center;
          }

          .platformDownloadButtonLeft {
            border-radius: 10%;
            width: 30px;
            height: 30px;
            border: none;
            background: rgba(60, 60, 60, 0.1);
            position: absolute;
          }

          .platformDownloadButton {
            border-radius: 50%;
            width: 30px;
            height: 30px;
            border: none;
            background: rgba(60, 60, 60, 0.1);
          }

          .platformDownloadButtonHover {
            border-radius: 50%;
            width: 30px;
            height: 30px;
            border: none;
            background-image: linear-gradient(to right, #1fc2c8, #9337d8 116%);
            color: white;
          }

          .platformIcon {
            font-size: 32px;
            margin-right: 10px;
          }

          p {
            font-style: normal;
            font-weight: normal;
            font-size: 14px;
            line-height: 21px;
            margin-top: 15px;
          }
        }

        .pwaCard:hover {
          box-shadow: 0 1px 8px 4px #9a989869;
        }

        #pwaMainCard {
          grid-column-start: span 2;
        }

        #pwaTeamsCard {
          grid-column-start: span 2;
        }

        // #pwaWindowsCard {
        //   #platformIcon {
        //     margin-right: 42px;
        //   }

        //   .pwaCardHeaderBlock {
        //     margin-right: 100px;
        //   }

        //   h2 {
        //     margin-right: -96px;
        //   }
        // }

        #windowsListItem {
          height: 100px;
        }

        li {
          display: flex;
          height: 84px;
          margin-bottom: 30px;

          span {
            margin-left: 19px;
            font-size: 14px;
            font-weight: normal;
            width: 476px;
          }

          #platformButtonBlock {
            display: flex;
            flex-direction: column;
            align-items: center;

            .platformIcon {
              font-size: 44px;
            }
          }
        }
      }
    }
  }
}

#topLabelBox {
  margin-bottom: 1em;
}

#bottomSection {
  display: none;

  #coolPWAs {
    padding-left: 10em;
    flex: 1;

    h2 {
      font-size: 36px;
      font-weight: bold;
    }

    p {
      width: 392px;
      font-size: 18px;
    }

    #iconGrid {
      display: grid;
      grid-template-columns: auto auto auto;
      width: 392px;
      margin-top: 36px;

      svg {
        font-size: 64px;
      }
    }
  }

  #bottomImageSection {
    flex: 1;
  }
}

.androidModalBody #extraSection p {
  color: grey;
  font-size: 14px;
}

.androidModalBody #extraSection #legacyDownloadButton, 
.androidModalBody #androidModalButtonSection #legacyDownloadButton, 
#legacyEdgeBetaDownloadButton, 
.legacyEdgeBetaOptionsButton {
  color: grey;
  font-size: 14px;
  background: transparent;
  padding-left: 0;
  border: none;
  margin-top: 1em;
}

#legacyDownloadButton {
  vertical-align: bottom;
}

#legacyEdgeBetaDownloadButton {
  vertical-align: bottom;
}

#legacyEdgeBetaDownloadButton #colorSpinner {
  transform: scale(0.5) translateY(-5px);
  max-height: 16px;
}

#legacyEdgeBetaDownloadButton, .legacyEdgeBetaOptionsButton {
  color: #9337d8;
  border: solid 1px;
  padding: 4px;
  margin-right: 4px;
}

.edgeBlock {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
}

.androidModalBody {
  width: 34em;
  background: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-left: 60px;
  padding-right: 60px;
  border-radius: 12px;
}

.androidModalBody.androidOptionsModalBody {
  width: 100%;
  align-items: start;
  padding-left: 0;
  max-height: 500px;
  overflow: auto;
}

/* On smaller screens, reduce the padding on modals */
@media (max-width: $media-screen-m) {
  .androidModalBody.androidOptionsModalBody {
    padding-left: 10px;
    padding-right: 10px;
  }
}

.androidModalBody.androidOptionsModalBody input {
  margin-bottom: 12px;
  border-color: #00000075;
}

.androidModalBody.androidOptionsModalBody label {
  display: block;
  margin-bottom: 10px;
}

.androidModalBody.androidOptionsModalBody + .modal-buttons {
  margin-top: 2em;
  margin-bottom: 2em;
}

.androidModalBody.androidOptionsModalBody .fa-info-circle {
  color: $color-muted;
  cursor: help;
}

.androidModalBody.androidOptionsModalBody a .fa-info-circle {
  cursor: pointer;
}

#closeAndroidPlatButton {
  top: 10px;
  border: none;
  float: right;
  height: 32px;
  background: #3c3c3c;
  color: white;
  border-radius: 50%;
  width: 32px;
  margin-top: 10px;
  margin-right: 10px;
  right: 10px;
  position: absolute;
  font-size: 14px;
}

.androidModalP {
  font-family: sans-serif;
  font-style: normal;
  font-weight: 600;
  font-size: 16px;
  line-height: 24px;
  letter-spacing: -0.02em;
  margin-top: 40px;
}

.androidModalP a {
  color: #9337d8;
}

.androidModalBody #androidModalSubText {
  color: #3c3c3c;
  display: block;
  margin-bottom: 2em;
  font-style: normal;
  font-weight: normal;
  font-size: 14px;
  line-height: 21px;
}

#androidModalButtonSection {
  display: flex;
  justify-content: space-around;
  width: 80%;
  margin-top: 1em;
  margin-bottom: 20px;
}

.androidDownloadButton {
  background: #9337d8;
  color: white;
  font-size: 14px;
  border-radius: 20px;
  width: 150px;
  height: 40px;
  padding-left: 20px;
  padding-right: 20px;
  font-family: sans-serif;
  font-style: normal;
  font-weight: 600;
  display: flex;
  justify-content: center;
  align-items: center;
  border: none;
}

.androidDownloadButton #installSpinner {
  margin-top: 4px;
}

.androidDownloadButton.webviewButton {
  width: 183px;
  background: #3c3c3c;
}

.androidDownloadButton:hover {
  cursor: pointer;
}

#androidPlatModal {
  background: transparent;
  position: fixed;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 99999;
  animation-name: opened;
  animation-duration: 250ms;
  border-radius: 4px;
  will-change: opacity transform;
}

.platModalBody #extraSection p {
  color: grey;
  font-size: 10px;
}

.platModalBody #extraSection #legacyDownloadButton {
  color: grey;
  font-size: 10px;
  background: transparent;
}

.platModalBody {
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  width: 34em;
  background: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-left: 60px;
  padding-right: 60px;
  border-radius: 12px;
  text-align: center;
}

.platModalForm {
  max-height: 800px;
  padding: 36px 60px;
  overflow: scroll;
  justify-content: space-between;
  align-content: space-between;

  .platModalField {
    width: 100%;
    text-align: left;
    flex: 0 0;

    &:not(:first-child) {
      margin-top: 14px;
    }

    label {
      h4 {
        font-size: 16px;
        line-height: 19px;

        color: #000000;
      }

      p {
        margin: 8px 0;
        font-size: 14px;
        line-height: 16px;
        color: #808080;
      }

      margin-bottom: 16px;
    }

    input,
    textarea {
      width: 100%;
      font-size: 14px;
      border-color: #f4f4f4;

      &:focus {
        border-color: #9337d8;
        outline: none;
      }

      &.error {
        border-color: #a80000;
      }
    }

    input {
      height: 35px;
      padding: 8px 8px 8px 0;
      border-width: 0 0 1px;
    }

    textarea {
      height: 120px;
      resize: none;

      &.short {
        height: 40px;
      }
    }

    .image-upload-loader {
      display: inline-block;
      margin-left: 8px;
      vertical-align: middle;
    }

    &.file-chooser {
      #uploadIconImage-color,
      #uploadIconImage-outline {
        font-size: 14px;
        line-height: 16px;
        text-align: center;
        display: inline-block;
        padding: 8px 32px;
        color: #000000;
        border: 1px solid #3c3d3e;

        &:focus {
          border-color: #9337d8;
        }
      }

      #upload-file-input-color,
      #upload-file-input-outline {
        display: none;
      }

      .file-description {
        display: inline-block;
        margin-left: 14px;
        font-size: 14px;
        line-height: 16px;
        color: #808080;
      }
    }
  }
}

.closeModalPlatButton {
  top: 10px;
  border: none;
  float: right;
  height: 32px;
  background: #3c3c3c;
  color: white;
  border-radius: 50%;
  width: 32px;
  margin-top: 10px;
  margin-right: 10px;
  right: 10px;
  position: absolute;
  font-size: 14px;
}

.platModalP {
  font-family: sans-serif;
  font-style: normal;
  font-weight: 600;
  font-size: 16px;
  line-height: 24px;
  letter-spacing: -0.02em;
  margin-top: 40px;
}

.platModalP a {
  color: #9337d8;
}

.platModalBody .platModalSubText {
  color: #3c3c3c;
  display: block;
  margin-bottom: 2em;
  font-style: normal;
  font-weight: normal;
  font-size: 14px;
  line-height: 21px;
}

.platModalButtonSection {
  display: flex;
  justify-content: space-around;
  width: 80%;
  margin-top: 1em;
  margin-bottom: 20px;
}

.platModalDownloadButton {
  background: #9337d8;
  color: white;
  font-size: 14px;
  border-radius: 20px;
  width: 150px;
  height: 40px;
  padding-left: 20px;
  padding-right: 20px;
  font-family: sans-serif;
  font-style: normal;
  font-weight: 600;
  display: flex;
  justify-content: center;
  align-items: center;
  border: none;
}

.platModalDownloadButton #installSpinner {
  height: 32px;
  margin-top: 4px !important;
}

.platModalDownloadButton.webviewButton {
  width: 183px;
  background: #3c3c3c;
}

.platModalDownloadButton:hover {
  cursor: pointer;
}

.platModal {
  background: transparent;
  position: fixed;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 99999;
  animation-name: opened;
  animation-duration: 250ms;
  border-radius: 4px;
  will-change: opacity transform;
}

.getIconButton {
  background: #8830d2;
  font-size: 12px;
  line-height: 14px;
  border: none;
  border-radius: 3px;
  color: white;
  padding: 4px 6px;
  width: 30%;
  float: left;
  text-align: center;
}

#teamsModal {
  align-items: baseline;
}

@keyframes opened {
  from {
    transform: scale(0.4, 0.4);
    opacity: 0.4;
  }

  to {
    transform: scale(1, 1);
    opacity: 1;
  }
}

.modalBackground {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: grey;
  opacity: 0.8;
  z-index: 98999;
}

.packageError {
  position: fixed;
  bottom: 2em;
  right: 2em;
  max-width: 350px;
  overflow: hidden;
  font-size: 0.875rem;
  background-color: rgba(255, 255, 255, 0.85);
  background-clip: padding-box;
  border: 1px solid rgba(0, 0, 0, 0.1);
  box-shadow: 0 0.25rem 0.75rem rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(10px);
  border-radius: 0.25rem;
  z-index: 999999;

  .errorHeader {
    display: flex;
    align-items: center;
    padding: 0.25rem 0.75rem;
    color: #6c757d;
    background-color: rgba(255, 255, 255, 0.85);
    border-bottom: 1px solid rgba(0, 0, 0, 0.05);

    .errorTitle {
      flex-grow: 1;
    }

    .closeBtn {
      opacity: 0.5;
      border: 0;
      background-color: transparent;
      padding: 0;
      font-size: 1.5rem;
      font-weight: bold;

      &:hover {
        color: #000;
        opacity: 0.75;
      }
    }
  }

  .errorBody {
    padding: 0 12px;
    max-height: 300px;
    overflow: auto;

    pre {
      font-family: $font-family-l3;
    }
  }

  .errorFooter {
    padding: 12px;
    display: flex;
    justify-content: flex-end;
    font-size: 90%;
    color: rgba(140, 140, 140, 0.18);
  }
}

@media (min-width: 1200px) {
  #main {
    height: 100vh;
  }

  #publishSideBySide {
    height: 100vh;
  }
}

@media (max-width: 920px) {
  #publishSideBySide {
    flex-direction: column;
  }

  #publishSideBySide #publishLeftSide #introContainer {
    margin-top: 20px;
    margin-left: 30px;
    margin-right: 30px;
  }
}

@media (max-width: 550px) {
  #publishSideBySide #publishRightSide #platformsListContainer ul {
    display: flex;
    flex-direction: column;
  }
}

#appGalleryModalBody #extraSection p {
  color: grey;
  font-size: 10px;
}

#appGalleryModalBody #extraSection #legacyDownloadButton {
  color: grey;
  font-size: 10px;
  background: transparent;
  padding-left: 0;
  border: none;
}

#appGalleryModalBody {
  width: 54em;
  background: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-left: 60px;
  padding-right: 60px;
  border-radius: 12px;
}

#appGalleryModalBody.appGalleryOptionsModalBody {
  width: 100%;
  align-items: start;
  padding-left: 0;
  max-height: 400px;
  overflow: auto;
}

#appGalleryModalBody.appGalleryOptionsModalBody input {
  margin-bottom: 12px;
  border-color: #00000075;
}

#appGalleryModalBody.appGalleryOptionsModalBody label {
  display: block;
  margin-bottom: 10px;
}

#appGalleryModalBody.appGalleryOptionsModalBody + .modal-buttons {
  margin-top: 2em;
  margin-bottom: 2em;
}

#appGalleryModalBody.appGalleryOptionsModalBody .fa-info-circle {
  color: $color-muted;
}

#closeAppGalleryPlatButton {
  top: 10px;
  border: none;
  float: right;
  height: 32px;
  background: #3c3c3c;
  color: white;
  border-radius: 50%;
  width: 32px;
  margin-top: 10px;
  margin-right: 10px;
  right: 10px;
  position: absolute;
  font-size: 14px;
}

#appGalleryModalP {
  font-family: sans-serif;
  font-style: normal;
  font-weight: 600;
  font-size: 16px;
  line-height: 24px;
  letter-spacing: -0.02em;
  margin-top: 40px;
}

#appGalleryModalP a {
  color: #9337d8;
}

#appGalleryModalBody #appGalleryModalSubText {
  color: #3c3c3c;
  display: block;
  margin-bottom: 2em;
  font-style: normal;
  font-weight: normal;
  font-size: 14px;
  line-height: 21px;
}

#appGalleryModalButtonSection {
  display: flex;
  justify-content: space-around;
  width: 80%;
  margin-top: 1em;
  margin-bottom: 20px;
}

.appGalleryDownloadButton {
  background: #9337d8;
  color: white;
  font-size: 14px;
  border-radius: 20px;
  width: 150px;
  height: 40px;
  padding-left: 20px;
  padding-right: 20px;
  font-family: sans-serif;
  font-style: normal;
  font-weight: 600;
  display: flex;
  justify-content: center;
  align-items: center;
  border: none;
  margin: 0 auto;
}

#appGalleryPlatModal {
  background: transparent;
  position: fixed;
  width: 100%;
  height: 80%;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 99999;
  animation-name: opened;
  animation-duration: 250ms;
  border-radius: 4px;
  will-change: opacity transform;
}

.appGalleryDownloadButton {
  background: linear-gradient(to right, #1fc2c8, #9337d8 116%);
  color: #ffffff;
  width: 300px;
}

.form-control-file {
  height: 16px;
  font-size: 12px;
  line-height: 14px;
}

.tab-content {
  padding: 20px 40px;
  border: solid 1px #cfcfcf;
  border-bottom-left-radius: 6px;
  border-bottom-right-radius: 6px;
}

.appGalleryDownloadText {
  text-align: center;
  font-size: 12px;
}

.appGalleryText {
  font-size: 11px;
  line-height: 14px;
}

.wizard-header {
  padding-top: 15px;
}
</style>
