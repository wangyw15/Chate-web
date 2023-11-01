<style>
body {
  margin: 0;
}

.app-container {
  display: flex;
  align-items: center;
  flex-direction: column;
  width: 100%;
}

.main-app {
  width: 100%;
}

@media screen and (min-width: 400px) {
  .main-app {
    width: 400px;
  }
}
</style>

<template>
  <div class="app-container">
    <el-container class="main-app">
        <el-header>
            <ClientOnly>
                <el-menu>
                    <el-menu-item index="0">
                        <h1 style="display: inline-block;">吃谷助手</h1>
                    </el-menu-item>
                </el-menu>
            </ClientOnly>
        </el-header>
        <el-main>
          <NuxtPage />
        </el-main>
    </el-container>
  </div>
</template>
