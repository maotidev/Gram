<template>
    <Modal :show="show" @close="showModalPost" :maxWidth="'5xl'">
        <div class="bg-white overflow-hidden shadow-none">
            <div class="grid grid-cols-3 min-w-full">

                <div class="col-span-2 w-full">
                    <img class="w-full max-w-full min-w-full"
                         :src="post.image_path"
                         :alt="post.description">
                </div>

                <div class="col-span-1 relative pl-4">
                    <header class="border-b border-grey-400">
                        <Link :href="'/profile/'+post.user.nick_name"
                           class="block cursor-pointer py-4 flex items-center text-sm outline-none focus:outline-none focus:border-gray-300 transition duration-150 ease-in-out">
                            <img
                                :src="post.user.profile_photo_url"
                                class="h-8 w-8 rounded-full object-cover"
                                :alt="post.user.nick_name"/>
                            <p class="block ml-2 font-bold">{{ post.user.nick_name }}</p>
                        </Link>
                    </header>

                    <div>
                        <div class="scroll" ref="scrollComments">
                            <div class="pt-1">
                                <Comments :comment="post.description" :nickName="post.user.nick_name"
                                          :urlImage="post.user.profile_photo_url"></Comments>
                            </div>
                            <Comments v-if="post.comments.length > 0" v-for="(item , index) in  post.comments"
                                      :key="index" :comment="item.comment" :nickName="item.user.nick_name"
                                      :urlImage="item.user.profile_photo_url"></Comments>
                            <div v-else class="w-100 text-center text-gray-500">No hay comentarios</div>
                        </div>
                    </div>

                    <div class="absolute bottom-0 left-0 right-0 pl-4">
                        <div class="pt-4">
                            <div class="mb-2">
                                <div class="flex items-center">

                                    <span @click="likeDisLike" class="mr-3 inline-flex items-center cursor-pointer">
                        <svg class="text-red-500 inline-block h-7 w-7"
                             :class="[post.likes.find(like => like.user_id ===$page.props.user.id) ? 'fill-current' : 'hover:fill-current']"
                             xmlns="http://www.w3.org/2000/svg" fill="none"
                             viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                  d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z"/>
                        </svg>
                    </span>

                                    <span class="mr-3 inline-flex items-center cursor-pointer">
                                <svg class="text-gray-700 inline-block h-7 w-7 " xmlns="http://www.w3.org/2000/svg"
                                     fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                          d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z"/>
                                </svg>
                            </span>
                                </div>
                                <span class="text-gray-600 text-sm font-bold">{{ post.countLikes }} Likes</span>
                            </div>
                            <span class="block ml-2 text-xs text-gray-600">{{
                                    getDifferenceTime(post.created_at)
                                }}</span>
                        </div>

                        <div class="pt-4 pb-1 pr-3">
                            <div class="flex items-start">
                                <input v-model="data.textComment"
                                       class="w-full resize-none outline-none appearance-none"
                                       aria-label="Agrega un comentario..." placeholder="Agrega un comentario..."
                                       autocomplete="off" autocorrect="off" style="height: 36px;"/>
                                <button v-if="data.textComment.length > 0"
                                        @click="comment($page.props.user.id)"
                                        class="mb-2 focus:outline-none border-none bg-transparent text-blue-600">
                                    Publicar
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </Modal>
</template>

<script setup>
import Comments from "./Comments";
import Modal from '@/Jetstream/Modal'
import {Head, Link} from '@inertiajs/inertia-vue3';
import moment from 'moment'
import {reactive, ref} from "vue";

const props = defineProps({
    post: '',
    show: false
})

const emit = defineEmits(['show'])

const data = reactive({
    textComment: ''
})

const scrollComments = ref(null)

let showModalPost = () => {
    emit('show')
}

let getDifferenceTime = (date) => {
    return moment(date).toNow(true)
}

let likeDisLike = async () => {
    const URL = `/like-post`
    const RPT = (await axios.post(URL, {post_id: props.post.id})).data
    props.post.likes = RPT.likes
    console.log(RPT.likes)
    if (RPT.likes.length > 0) {
        console.log('Increment')
        props.post.countLikes++
    } else {
        console.log('Decrement')
        props.post.countLikes--
    }

}

let comment = async (userId) => {
    const URL = `/comment`

    const RPT = (await axios.post(URL, {
        post_id: props.post.id,
        user_id: userId,
        comment: data.textComment
    })).data

    props.post.comments.push(RPT)
    props.post.countComments++
    data.textComment = ''
    scrollBottom()


}

let scrollBottom = () => {
    setTimeout(() => {
        scrollComments.value.scrollTop = scrollComments.value.scrollHeight - scrollComments.value.clientHeight
    }, 50)
}

</script>

<style>

.scroll {
    height: 185px;
    overflow-y: auto;
}

.scroll::-webkit-scrollbar {
    width: 12px;
}

.scroll::-webkit-scrollbar-track {
    -webkit-box-shadow: inset 0 0 2px rgba(0, 0, 0, 0.3);
    border-radius: 10px;
}

.scroll::-webkit-scrollbar-thumb {
    -webkit-box-shadow: inset 0 0 2px rgba(0, 0, 0, 0.5);
    border-radius: 10px;
}


</style>
