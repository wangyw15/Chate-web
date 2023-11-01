<template>
  <el-container>
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
</template>
